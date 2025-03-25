#### nestjs实现静态文件服务及文件上传功能

1. 开启静态文件服务, 同时开启cors允许跨域访问, `需要注意的是,静态目录和打包后目录不同,生产环境部署时需要进行处理`

   ```ts
   // app.module.ts
   import { ServeStaticModule } from '@nestjs/serve-static';
   
   @Module({
     imports: [ ServeStaticModule.forRoot({
         rootPath: join(__dirname, '..', 'static'), // 静态文件的根路径
         serveRoot: '/static', // 设置 URL 路径前缀
         // 如果需要设置 CORS，可以在这里自定义 headers
         serveStaticOptions: {
           setHeaders: (res: any) => {
             res.setHeader('Access-Control-Allow-Origin', '*'); // 允许所有源访问
             res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE'); // 允许的请求方法
             res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Accept'); // 允许的请求头
           }
         }
    })
    ]})
   ```

2. 配置文件上传目录及加工逻辑

   ```ts
   //  multer.config.ts
   import { diskStorage } from 'multer';
   import { join } from 'path';
   
   export const multerConfig = {
     storage: diskStorage({
       // 文件保存的路径 使用根路径 rootPath   //  此处 代码会被打包进dist目录 而非开发时的根目录   所以需要使用join(__dirname, '..', '..', 'static')
       destination: join(__dirname, '..', '..', 'static/avatar'),
       filename: (req, file, cb) => {
         // 文件名使用时间戳和原始文件扩展名 
         const fileExt = file.mimetype.split('/')[1];
         const fileName = `${Date.now()}.${fileExt}`;
         cb(null, fileName);
       },
     }),
     limits: { fileSize: 5 * 1024 * 1024 }, // 限制文件大小为 5MB
   };
   ```

3. 相应模块的控制器里使用, 文件拦截器会按`multerConfig`配置,自动存储好文件返回相应信息

   ```ts
   import { Body, Post, Req, UploadedFile, UseInterceptors } from '@nestjs/common';
   import { Express } from 'express';
   import { FileInterceptor } from '@nestjs/platform-express';
   import { multerConfig } from '@/staticfile/multer.config';
   
   @Post('upload/avatar')
     @UseInterceptors(FileInterceptor('file', multerConfig))
     uploadFile(@UploadedFile() file: Express.Multer.File, @Body() body: any, @Req() req: any) {
       return this.userService.updateAvatar(file.filename, +body.id);
     }
   ```

   