前后端双token的实现

**基本原理: **

> 1. 主要原理同上一篇, 缺陷: 无法实时下线用户,因为有短token的窗口期
> 2. 登录时后端颁发长短期的双token(jwt+refresh), jwt进行接口校验, 失效时用refresh获取新的jwt及新的refresh,这次jwtid存的是rtToken
> 3. 差异点: accesstoken依旧返给前端, rtToken放入cookies
> 4. 校验需要2个guard结合,常规接口依旧使用jwtGuard, 换取新token的接口单独使用refreshGuard

1. 常规配置简略版

   ```ts
     @Post('rt/login')
     @ApiOperation({ summary: '用户登录(refreshToken版本)' })
     @UseGuards(CaptchaGuard)
     rtLogin(@Body() loginInfo: LoginInfoDto, @Res({ passthrough: true }) res: Response) {
       return this.authService.rtLogin(loginInfo, res);
     }
   //  登录成功后签发token  需要传递res 设置cookies
   const { accessToken } = await this.rtTokenService.signToken(id, { username, phone, id }, res);
   
   //  前端请求此登录接口时需要设置withCredentials不然不会自动设置rt
   ```

2. TokenService核心代码

   ```ts
   // 其他同上一篇  唯一区别 双token
    const [accessToken, refreshToken] = await Promise.all([
         await this.jwt.signAsync({ sub: userId, ...extraPayload }, { expiresIn: this.JWT_EXPIRES_TIME, secret: this.JWT_SECRET }),
         await this.jwt.signAsync({ id: userId }, { expiresIn: this.JWT_REFRESH_EXPIRES_TIME, jwtid: jti, secret: this.JWT_REFRESH_SECRET }),
       ]);
   //  签发后还需要 设置cookies
   setRtCookie(res: Response, refreshToken: string) {
       res.cookie('rt', refreshToken, {
         httpOnly: true,
         secure: !true,
         sameSite: 'lax',
         path: '/',
         maxAge: 15 * 24 * 60 * 60 * 1000,
       });
     }
   ```

3. JwtRefreshStrategy配置

   ```ts
   export class JwtRefreshStrategy extends PassportStrategy(Strategy, 'jwt-refresh') {
     constructor(configService: ConfigService) {
       super({
         jwtFromRequest: ExtractJwt.fromExtractors([(req: Request) => req?.cookies?.rt || null]), // 从请求体中取refreshToken
         // 密匙如果对不上 会直接报错 user返回false
         secretOrKey: configService.get<string>('JWT_REFRESH_SECRET'),
       });
     }
     validate(payload: any) {
       return payload;
     }
   }
   ```

4. refreshGuard和上一篇JwtGuard一样, 然后新的JwtGuard改为校验`await super.canActivate(context)`即可,因为短的jwtToken只要僬侥有效性即可

   ```ts
   // 注意guard和strategy的策略名要一致
   export class JwtStrategy extends PassportStrategy(Strategy) {}
   export class JwtAuthGuard extends AuthGuard('jwt') {}
   // 当继承PassportStrategy或AuthGuard时不传参,默认就是'jwt'
   export class JwtRefreshStrategy extends PassportStrategy(Strategy, 'jwt-refresh') {}
   export class JwtRefreshAuthGuard extends AuthGuard('jwt-refresh') {}
   ```

5. 前端使用axios无感刷新,主要原理是在拦截器请求新token并重发之前的请求,同时配置`withCredentials`携带cookies,后端的JwtRefreshStrategy会从cookie读取, 当然从header也行, 只要前后端约定好统一的传递方式即可

   ```ts
   axiosInstance.interceptors.response.use(
     (res: AxiosResponse) =>  res,
     async (error: AxiosError) => {
       const original = await slientTokenRefresh(error?.status || 0, error?.config)
       if (original) {
         const res = await axiosInstance(original)
         return { data: res } //  为什么要包裹?  因为后续还有拦截器在 进行逻辑判断  根据你的实际情况返回
       }
     })
   ```

6. 单独创建一个axios实例,避免走相同的拦截器

   ```ts
   
   import axios, { AxiosRequestConfig } from 'axios'
   import { useUserStoreWithOut } from '@/store/modules/user'
   export const PATH_URL = import.meta.env.VITE_API_BASE_PATH
   
   export const refreshApi = axios.create({
     baseURL: PATH_URL, // 或你的后端网关
     withCredentials: true // 让浏览器带上 HttpOnly RT
   })
   
   export type RefreshFn = () => Promise<string> // 返回新的 accessToken
   export class TokenRefresher {
     private running: Promise<string> | null = null
     private lastToken: string | null = null
     private lastAt = 0
     constructor(
       private readonly refreshFn: RefreshFn,
       private readonly graceMs = 300 // 可调的缓冲窗口
     ) {}
     /** 尝试刷新：并发去重；成功返回新 token；失败原样抛错（让上层处理） */
     tryRefresh(): Promise<string> {
       // 已有刷新在进行 → 直接复用同一个 Promise
       if (this.running) return this.running
   
       // 刚刷过（极短时间内的追击 401）→ 直接返回新 token，避免二次刷新
       if (this.lastToken && Date.now() - this.lastAt < this.graceMs) {
         console.log('===========缓存====')
         return Promise.resolve(this.lastToken)
       }
       this.running = this.refreshFn()
         .then((token) => {
           this.lastToken = token // 记录最近一次成功
           this.lastAt = Date.now()
           return token
         })
         .finally(() => {
           // 无论成功失败都重置，下一次401再触发
           this.running = null
         })
       return this.running
     }
   }
   
   // 和后端约定：POST /auth/refresh → { accessToken: string }
   export const refreshFn = async (): Promise<string> => {
     const res = await refreshApi.post('/auth/refresh')
     const { access_token } = res?.data?.data || {}
     if (!access_token) throw new Error('No accessToken from /auth/refresh')
     return access_token as string
   }
   
   // singleton.ts   //  包裹 class 实现  单例模式
   export function singleton<T>(key: string, create: () => T): T {
     const g = globalThis as any
     return (g[key] ??= create())
   }
   
   //  开发阶段 有bug 热更新 可能会导致2次new  从而多次请求
   const refresher = singleton('refresher', () => new TokenRefresher(refreshFn as RefreshFn, 3000))
   export const slientTokenRefresh = async (
     status: number,
     original: (AxiosRequestConfig & { _retry?: boolean }) | undefined
   ) => {
     const userStore = useUserStoreWithOut()
   
     //  这里的if作用进行版本token 快照比对   假如原始token和现有的不一致  说明之前已经请求过了  所以直接使用store的就行了
     // const originalToken = original?.headers && original.headers['Authorization']
     // console.log('originalToken', originalToken)
     // if (userStore.getToken && original?.headers && originalToken !== 'bearer ' + userStore.getToken) {
     //
     //   console.log('originalToken----=============-----------', userStore.getToken)
     //   original.headers['Authorization'] = 'bearer ' + userStore.getToken
     //   return original
     // }
   
     const isATExpired = status === 401 || status === 419
     if (isATExpired && original && !original._retry) {
       original._retry = true // 防死循环
       try {
         // ✨ 只做刷新本职；内部自动并发去重
         const newToken = await refresher.tryRefresh()
         userStore.setToken(newToken)
         // 把结果交给你原有体系：更新存储 + 重放这一个请求
         original.headers = original.headers ?? {}
         original.headers['Authorization'] = 'bearer ' + newToken
         // ★ 关键：return 重放 Promise，让调用方拿到“各自”的数据
         return original
       } catch (error: any) {
         if (error?.status === 401) userStore.logout()
         return false
         // 刷新失败 → 交由你原有逻辑统一处理（比如清理状态、跳登录）
         // 这里不弹“登录过期”的 toast，留给下面的统一错误分支处理
       }
     }
   }
   
   ```

   