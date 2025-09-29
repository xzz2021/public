nestjs使用jwtToken实现多点登录限制及主动失效token

**基本原理:**

> 1. token签发时分别在redis存入jwtid和jwtid的数组, 同时用数组剔除最老的jwtid并拉黑, 然后存入新的jwtid, 数组长度决定允许多少端点登录
> 2. guard层对解析出的userid和jwtid进行合法性校验
> 3. 通过对jwtid和数组的控制可以实现自动或主动失效

1. 常规配置简略版

   ```ts
     @Post('login')
     @ApiOperation({ summary: '用户登录' })
     @UseGuards(CaptchaGuard)
     login(@Body() loginInfo: LoginInfoDto) {
       return this.authService.login(loginInfo);
     }
   
   //  登录成功后签发token
   const accessToken = await this.tokenService.signToken(id, { username, phone, id});
   
   ```

2. TokenService简略版

   ```ts
   
   export class TokenService {
     private readonly cfg: SessionConfig = {
       lockTtlMs: 400, // 锁的 TTL
       lockMaxWaitMs: 300, // 最多等待拿锁 300ms
     };
     private maxSessions: number;
     private JWT_SECRET: string;
     private JWT_EXPIRES_TIME: number;
     private readonly redis: Redis;
     constructor(
       private readonly jwt: JwtService,
       private readonly redisService: RedisService,
       private readonly configService: ConfigService,
     ) {
       this.redis = this.redisService.getOrThrow();
           // ssoCount控制允许端点数量
       const ssoCount = this.configService.get<string>('SSO_COUNT') || 2; 
       this.maxSessions = Number(ssoCount);
       this.JWT_SECRET = this.configService.get<string>('JWT_SECRET') || '';
       this.JWT_EXPIRES_TIME = Number(this.configService.get<string>('JWT_EXPIRES_TIME') || 60 * 60 * 24 * 7);
     }
   
     // —— Key helpers ——
     private listKey(userId: number) {return `user:sessions:${userId}`; }
     private jtiKey(jti: string) {return `session:exp:${jti}`;}
     private blKey(jti: string) {return `jwt:blacklist:${jti}`;}
     private lockKey(userId: number) { return `user:sessions:lock:${userId}`;}
   
     // —— 轻量互斥锁（只用 cache-manager 的 get/set/del） ——
     // 1. 操作串行化  2. 减少并发写入导致的覆盖/错删等竞态问题 3. 减少“羊群效应”
     private async withUserListLock<T>(userId: number, fn: () => Promise<T>): Promise<T> {
       const lockKey = this.lockKey(userId);
       const token = randomUUID();
       const ttlMs = this.cfg.lockTtlMs!;
       const maxWait = this.cfg.lockMaxWaitMs!;
       const start = Date.now();
       // 尝试获取锁（尽量减少并发写冲突）
       while (true) {
         const existing = await this.redis.get(lockKey);
         if (!existing) {
           // 没锁 → 设置；cache-manager 不支持 NX，只能“尽快设置+短TTL”
           await this.redis.set(lockKey, token, 'PX', ttlMs);
           // 双检，降低竞争窗口
           const confirm = await this.redis.get(lockKey);
           if (confirm === token) break;
         }
         if (Date.now() - start > maxWait) break; // 超时放弃加锁，直接走非互斥路径（尽力而为）
         await new Promise(r => setTimeout(r, Math.floor(10 + Math.random() * 20))); // 随机抖动
       }
       try {
         const res = await fn();
         return res;
       } finally {
         const v = await this.redis.get(lockKey);
         if (v === token) {
           await this.redis.del(lockKey);
         }
       }
     }
     // 从缓存读取用户会话列表（数组形式）
     private async loadList(userId: number): Promise<string[]> {
       const raw = await this.redis.get(this.listKey(userId));
       if (!raw) return [];
       try {
         const arr = JSON.parse(raw);
         return Array.isArray(arr) ? arr : [];
       } catch {
         return [];
       }
     }
     // 持久化用户会话列表
     private async saveList(userId: number, list: string[]) {
       // 永久有效
       await this.redis.set(this.listKey(userId), JSON.stringify(list));
     }
     /** 签发会话（含并发限制/逐出旧会话/黑名单） */
     async issue(userId: number, extraPayload: Record<string, any> = {}) {
       const jti = randomUUID();
       const nowSec = Math.floor(Date.now() / 1000);
       const exp = nowSec + this.JWT_EXPIRES_TIME;
       const token = await this.jwt.signAsync({ sub: userId, ...extraPayload }, { expiresIn: this.JWT_EXPIRES_TIME, jwtid: jti });
       // 记录 jti 的 exp（撤销时可直接查）
       await this.redis.set(this.jtiKey(jti), String(exp), 'EX', this.JWT_EXPIRES_TIME);
       // 修改用户的会话列表（尽量加锁，避免并发覆盖）
       await this.withUserListLock(userId, async () => {
         let list = await this.loadList(userId);
         // 新的放前面；去重（如果同 jti 不应出现；保险起见）
         list = [jti, ...list.filter(x => x !== jti)];
         // 超出上限 → 逐出尾部旧会话
         const evicted: string[] = list.length > this.maxSessions ? list.slice(this.maxSessions) : [];
         list = list.slice(0, this.maxSessions);
         await this.saveList(userId, list);
         // 把被逐出的 jti 拉黑
         for (const oldJti of evicted) {
           const expStr = await this.redis.get(this.jtiKey(oldJti));
           const oldExp = expStr ? Number(expStr) : 0;
           await this.blacklistByJti(oldJti, oldExp);
         }
       });
       return { jti, exp, token };
     }
   
     /** 撤销单个会话 */
     async revoke(userId: number, jti: string) {
       await this.withUserListLock(userId, async () => {
         const list = await this.loadList(userId);
         const newList = list.filter(x => x !== jti);
         if (newList.length !== list.length) {
           await this.saveList(userId, newList);
         }
       });
       const expStr = await this.redis.get(this.jtiKey(jti));
       const exp = expStr ? Number(expStr) : 0;
       await this.blacklistByJti(jti, exp);
     }
   
     /** 撤销全部会话，或保留某个 jti */
     async revokeAll(userId: number, exceptJti?: string) {
       let toRevoke: string[] = [];
       await this.withUserListLock(userId, async () => {
         const list = await this.loadList(userId);
         toRevoke = exceptJti ? list.filter(x => x !== exceptJti) : list;
         const newList = exceptJti ? [exceptJti] : [];
         await this.saveList(userId, newList);
       });
   
       for (const jti of toRevoke) {
         const expStr = await this.redis.get(this.jtiKey(jti));
         const exp = expStr ? Number(expStr) : 0;
         await this.blacklistByJti(jti, exp);
       }
     }
     /** jti 是否已拉黑 */
     async isBlacklisted(jti: string): Promise<boolean> {
       const v = await this.redis.get(this.blKey(jti));
       return v === '1';
     }
     /** 拉黑到过期为止（exp 过了就跳过） */
     async blacklistByJti(jti: string, exp: number) {
       const ttlSec = exp - Math.floor(Date.now() / 1000);
       if (ttlSec > 0) {
         await this.redis.set(this.blKey(jti), 1, 'EX', ttlSec);
       }
     }
     /** 获取当前用户会话列表（调试/管理用途） */
     async listSessions(userId: number) {
       return this.loadList(userId);
     }
     // 登录通过后（账号密码已校验）
     async signToken(userId: number, extraPayload = {}) {
       const { token } = await this.issue(userId, extraPayload);
       return token;
     }
     async logout(userId: number, jti: string) {
       await this.revoke(userId, jti);
       return { ok: true };
     }
     async kickOthers(userId: number, currentJti?: string) {
       await this.revokeAll(userId, currentJti);
       return { ok: true };
     }
   }
   ```

3. JwtStrategy配置

   ```ts
   export class JwtStrategy extends PassportStrategy(Strategy) {
     constructor(readonly configService: ConfigService) {
       super({
         jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
         ignoreExpiration: false,
         secretOrKey: configService.get<string>('JWT_SECRET'),
       });
     }
     validate(payload: any) {
       return payload;
     }
   }
   ```

4. guard守卫校验

   ```ts
   export class JwtAuthGuard extends AuthGuard('jwt') {
     constructor(
       private reflector: Reflector,
       private readonly tokenService: TokenService,
     ) {super();}
   
     async canActivate(context: ExecutionContext): Promise<boolean> {
       const request = context.switchToHttp().getRequest();
       // 允许对 `/static/` 开头的资源访问
       if (request.url.startsWith('/static/')) return true; 
       const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [context.getHandler(), context.getClass()]);
       if (isPublic)  return true;
           // 先跑 JWT 校验（签名/过期/解析 payload）
       const ok = (await super.canActivate(context)) as boolean;
       if (!ok) return false;
       const user = request.user;
       const userId = user?.sub as number;
       const jti = user?.jti as string;
       if (!userId || !jti) {
         throw new UnauthorizedException('Invalid token payload');
       }
       // 1) 黑名单校验：被撤销/踢下线的会话直接 401
       if (await this.tokenService.isBlacklisted(jti)) {
         throw new UnauthorizedException('Token 已失效');
       }
       // 2) 仍在该用户的会话列表中（避免已被逐出的旧会话继续访问）
       const list = await this.tokenService.listSessions(userId);
       if (!list.includes(jti)) {
         throw new UnauthorizedException('token not active');
       }
       return true;
     }
   
   }
   ```

   

