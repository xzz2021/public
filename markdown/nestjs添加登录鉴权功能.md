#### nestjs添加登录鉴权功能

> ##### 方案一, 仅使用jwt完成基础功能

1. 生成auth模块, 安装@nestjs/jwt

   ```bash
    nest generate resource auth --no-spec
    pnpm add  @nestjs/jwt
    # @nestjs/jwt 生成jwt的token  
   ```

   将登录接口signIn写在AuthService中

   ```ts
   
   import { Injectable, UnauthorizedException } from '@nestjs/common';
   import { UsersService } from '../users/users.service';
   import { JwtService } from '@nestjs/jwt';
   
   @Injectable()
   export class AuthService {
     constructor(private usersService: UsersService,private jwtService: JwtService) {}
     async signIn(username: string,pass: string): Promise<{ access_token: string }> {
       const user = await this.usersService.findOne(username);
       if (user?.password !== pass) {
         throw new UnauthorizedException();
       }
       const payload = { sub: user.userId, username: user.username };
       return {
         access_token: await this.jwtService.signAsync(payload),
       };
     }
   }
   ```

2. AutModule中注册为全局模块

   ```ts
   import { Module } from '@nestjs/common';
   import { AuthService } from './auth.service';
   import { UsersModule } from '../users/users.module';
   import { JwtModule } from '@nestjs/jwt';
   import { AuthController } from './auth.controller';
   import { jwtConstants } from './constants';
   
   @Module({
     imports: [
       UsersModule,
       JwtModule.register({
         global: true,
         secret: jwtConstants.secret,
         signOptions: { expiresIn: '60s' },
       }),
     ],
     providers: [AuthService],
     controllers: [AuthController],
     exports: [AuthService],
   })
   export class AuthModule {}
   ```

3. 生成AuthGuard注册为保护端点, 在接口上使用`@UseGuards(AuthGuard)`修饰

   ```ts
   import {CanActivate,ExecutionContext,Injectable,UnauthorizedException} from '@nestjs/common';
   import { JwtService } from '@nestjs/jwt';
   import { jwtConstants } from './constants';
   import { Request } from 'express';
   
   @Injectable()
   export class AuthGuard implements CanActivate {
     constructor(private jwtService: JwtService) {}
     async canActivate(context: ExecutionContext): Promise<boolean> {
        const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
         context.getHandler(),
         context.getClass(),
       ]);
       if (isPublic) return true  //  如果是公开接口直接放行
        
       const request = context.switchToHttp().getRequest();
       const token = this.extractTokenFromHeader(request);
       if (!token) {throw new UnauthorizedException();}
       try {
         const payload = await this.jwtService.verifyAsync(token,{secret:jwtConstants.secret});
         request['user'] = payload;
       } catch {
         throw new UnauthorizedException();
       }
       return true;
     }
     private extractTokenFromHeader(request: Request): string | undefined {
       const [type, token] = request.headers.authorization?.split(' ') ?? [];
       return type === 'Bearer' ? token : undefined;
     }
   }
   ```

4. 将AuthGuard在AppModule中注册为全局防护, Nestjs 将自动将`AuthGuard`绑定到所有端点(接口)

   ```ts
   providers: [ { provide: APP_GUARD, useClass: AuthGuard }]
   ```

   例外接口需要使用`@public()`修饰, 创建自定义装饰器

   ```ts
   import { SetMetadata } from '@nestjs/common';
   
   export const IS_PUBLIC_KEY = 'isPublic';
   export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
   ```

> ##### 方案二, 使用passport身份验证库

1. passport集成了各种校验方式, jwt, OAuth 2.0, 类似接入微信等三方平台免注册一键登录

2. 简单理解passport-local基于 `会话(Session)` 的认证方式, 适用于 SSR(服务端渲染),  **`LocalAuthGuard`**：用户登录时的凭证验证(用户名和密码),它用于 **初次登录认证**,  **`JwtAuthGuard`**：验证已登录用户的身份, 用于 **保护需要认证的路由**。初次账密登录,使用LocalAuthGuard校验会把查询到的user挂载到req.user, jwt同理, 如果不使用passport-local也可以在登录接口里自行处理, 以下代码只使用`passport-jwt`

3. 使用`JWT`, 安装依赖库, 生成auth模块

   ```bash
   pnpm add  @nestjs/jwt @nestjs/passport passport-jwt
   pnpm add  @types/passport-jwt  -D
   nest generate resource auth --no-spec
   ```

4. 将登录和注册都放在auth模块中, 添加`AuthService`

   ```ts
   import { Injectable } from '@nestjs/common';
   import { JwtService } from '@nestjs/jwt';
   import { PrismaService as pgService } from '@/prisma/prisma.service';
   import * as bcrypt from 'bcrypt';
   import { LoginInfo, RegisterInfo } from './types';
   
   @Injectable()
   export class AuthService {
     constructor(
       private readonly pgService: pgService,
       private readonly jwtService: JwtService,
     ) {}
     private readonly SALT_ROUNDS = 10; // 盐的轮数
   
     async create(createUserDto: RegisterInfo) {
       const hashedPassword = await bcrypt.hash(createUserDto.password, this.SALT_ROUNDS);
       // 应该要先查询下手机号是否存在,  存在 抛出异常提示
       const user = await this.pgService.user.findUnique({where: {phone: createUserDto.phone}});
       if (user) {
         // throw new Error('手机号已存在');
         return { code: 0, message: '手机号已存在' };
       }
       try {
         	const res = await this.pgService.user.create({
           data: {...createUserDto,password: hashedPassword},
         });
         return { code: 200, message: '注册成功', data: res };
       } catch (error) {
         console.log(error)  //  错误  抛出异常
         const { code, sqlMessage } = error;
         return { code, message: sqlMessage };
       }
     }
     async login(loginInfo: LoginInfo) {
       const user = await this.pgService.user.findUnique({
         where: { phone: loginInfo.phone },
       });
       if (!user) {return { code: 0, message: '用户不存在' }}
         //  比较时为什么不需要盐?因为盐在hash密码本身中
       const isPasswordValid = await bcrypt.compare(loginInfo.password, user.password);
       if (!isPasswordValid) {return { code: 0, message: '密码错误' }}
       // eslint-disable-next-line @typescript-eslint/no-unused-vars
       const { password, ...result } = user;
       const payload = { username: result.username, phone: result.phone, sub: result.id };
       return { code: 200, message: '登录成功', userinfo: result, access_token: this.jwtService.sign(payload) };
     }
   }
   ```

5. 添加`jwt-strategy`

   ```ts
   import { ExtractJwt, Strategy } from 'passport-jwt';
   import { PassportStrategy } from '@nestjs/passport';
   import { Injectable } from '@nestjs/common';
   
   @Injectable()
   export class JwtStrategy extends PassportStrategy(Strategy) {
     constructor() {
       super({
         jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
         ignoreExpiration: false,
         secretOrKey: `${process.env.JWT_SECRET}`,
       });
     }
     async validate(payload: any) {
       // 此处用于返回需要挂载到 所有走jwtguard的接口中 使用 @Request  req.user  的数据
       return { userId: payload.sub, username: payload.username };
     }
   }
   ```

6. 添加`AuthGuard`

   ```ts
   import { ExecutionContext, Injectable } from '@nestjs/common';
   import { Reflector } from '@nestjs/core';
   import { AuthGuard } from '@nestjs/passport';
   import { IS_PUBLIC_KEY } from './public';
   
   @Injectable()
   export class JwtAuthGuard extends AuthGuard('jwt') {
     constructor(private reflector: Reflector) {
       super();
     }
     canActivate(context: ExecutionContext) {
       const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
         context.getHandler(),
         context.getClass(),
       ]);
       if (isPublic) {
         return true;
       }
       return super.canActivate(context);
     }
   }
   ```

7. 添加`AuthModule`

   ```ts
   import { Module } from '@nestjs/common';
   import { AuthService } from './auth.service';
   import { JwtStrategy } from './jwt.strategy';
   import { UsersModule } from '../users/users.module';
   import { PassportModule } from '@nestjs/passport';
   import { JwtModule } from '@nestjs/jwt';
   import { jwtConstants } from './constants';
   
   @Module({
     imports: [ UsersModule, PassportModule,
       JwtModule.register({
         secret: jwtConstants.secret,
         signOptions: { expiresIn: '60s' },
       }),
     ],
     providers: [AuthService, JwtStrategy],
     exports: [AuthService],
   })
   export class AuthModule {}
   ```

8. 在需要的模块内引入

   ```ts
   import { Controller, Request, Post, UseGuards } from '@nestjs/common';
   import { LocalAuthGuard } from './auth/local-auth.guard';
   import { AuthService } from './auth/auth.service';
   
   @Controller()
   export class AppController {
     constructor(private authService: AuthService) {}
   
     @UseGuards(JwtAuthGuard) //???
     @Post('auth/login')
     async login(@Request() req) {
       return this.authService.login(req.user);
     }
   }
   ```

9. 全局挂载

   ```ts
    providers: [AppService, { provide: APP_GUARD, useClass: JwtAuthGuard }]
   ```

10. 公开接口放行, 使用@public装饰器, 同方案一4













