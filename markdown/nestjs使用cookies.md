nestjs使用cookies

1. 正常的数据都是nestjs接管了响应的, 所以可以直接return,而要设置cookie需要手动调用res

2. 首先全局设置允许cookies携带跨域

   ```ts
   import cookieParser from 'cookie-parser';
   
     app.use(cookieParser());
     app.enableCors({
       origin: ['http://localhost:4000'],
       credentials: true,
     });
   ```

3. controller使用装饰器,传递Response

   ```ts
   import { Response } from 'express';
     
   @Post('login')
     login(@Body() loginInfo: LoginInfoDto, @Res() res: Response) {
       return this.authService.login(loginInfo, res);
     }
   ```

4. 主逻辑代码使用,设定并返回数据

   ```ts
   res.cookie('rt', refreshToken, {
           httpOnly: true,
           secure: !true,
           sameSite: 'lax',
           path: '/',
           maxAge: 15 * 24 * 60 * 60 * 1000,
         });
   return res.status(200).json({ accessToken });
   ```

5. 如果希望可以直接return返回数据,需要传递`{ passthrough: true }`参数

   ```ts
   @Res({ passthrough: true })  //  让 Nest 继续负责序列化（JSON）
   ```

6. 以上方案二选一, 不然前端调用接口就会一直处于pending状态,等待数据返回