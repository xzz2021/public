#### prisma的使用

1. 初始化安装

   ```cmd
   
   pnpm add prisma typescript ts-node @types/node --save-dev
   
   npx tsc --init
   ```

2. 生成prisma初始化文件

   ```cmd
   npx prisma init
   # 此命令会自动生成schema.prisma配置文件 及 .env文件
   ```

   初始配置

   ```js
   generator client {
     provider = "prisma-client-js"
   }
   
   datasource db {
     provider = "postgresql"
     url      = env("DATABASE_URL")
   }
   // .env
   DATABASE_URL="postgresql://johndoe:randompassword@localhost:5432/mydb?schema=public"
   ```

3. 定义模型model, 执行命令,没有内容的新数据库可以使用

   ```cmd
   npx prisma migrate dev --name init
   # 数据模型会映射到数据库,自动生成表,默认读取/prisma/schema.prisma文件,
   # 1.为迁移创建新的 SQL 迁移文件, 2.将迁移应用于数据库
   # 但是如果是已有数据库, 模型定义不全会清除其他表格
   ```

4. 所以最好先自动根据已有数据库生成模型定义

   ```cmd
   npx prisma db pull
   # 假如是新增或修改了模型数据, 比如新增了一张表, 可以执行更新命令
   prisma db push
   ```

5. 安装使用并生成 Prisma 客户端

   ```cmd
   pnpm add @prisma/client
   # 生成 Prisma 客户端库
   npx prisma generate
   ```

6. 执行种子数据的写入(可以用于创建数据库初始化数据), 创建种子文件

   ```ts
   // seed.ts     放入prisma目录下
   
   import { PrismaClient } from '@prisma/client';
   
   // initialize Prisma Client
   const prisma = new PrismaClient();
   
   async function main() {
     // create two dummy articles
     const post1 = await prisma.test666.create({
       data: {
         title: 'Prisma Adds Support for MongoDB',
         authorId: 1,
         published: false,
       },
     });
   
     console.log({ post1 });
   }
   
   // execute the main function
   main()
     .catch((e) => {
       console.error(e);
       process.exit(1);
     })
     .finally(async () => {
       // close Prisma Client at the end
       await prisma.$disconnect();
     });
   
   ```

   添加配置及执行命令

   ```js
   // package.json 添加配置 
   "prisma": {
       "seed": "ts-node prisma/seed.ts"
     }
   // 执行写入命令
   npx prisma db seed
   ```

7. 上面方法适用任何node环境及其框架

### nestjs中使用prisma

> 本质上prisma是一个可以独立运行的orm库,导入其client就能操作数据库

1. 生成prisma模块

   ```cmd
   nest generate module prisma
   nest generate service prisma
   ```

2. 定义module文件

   ```ts
   //  prisma.module.ts
   import { Module } from '@nestjs/common';
   import { PrismaService } from './prisma.service';
   @Module({
     providers: [PrismaService],
     exports: [PrismaService],
   })
   export class PrismaModule {}
   ```

3. 定义service文件

   ```ts
   //   prisma.service.ts
   //  在只有连接一个数据库的情况下，prisma会自动生成默认数据库类型的client, 
   //  放在node_modules包内部的@prisma/client目录
   import { Injectable } from '@nestjs/common';
   import { PrismaClient } from '@prisma/client';  // 所以此处等于默认引入相应数据库的client
   @Injectable()
   export class PrismaService extends PrismaClient {}
   ```

4. 在全局或其他需要使用的模块引入module

   ```js
   // app.module.ts
   import { Module } from '@nestjs/common';
   import { AppController } from './app.controller';
   import { AppService } from './app.service';
   import { PrismaModule } from './prisma/prisma.module';
   
   @Module({
     imports: [PrismaModule],
     controllers: [AppController],
     providers: [AppService],
   })
   export class AppModule {}
   ```

5. 在其他service文件中使用prisma

   ```js
   // app.service.ts
   import { Injectable } from '@nestjs/common';
   import { PrismaService } from 'src/prisma/prisma.service';
   @Injectable()
   export class AppService {
     constructor(private prisma: PrismaService) {}
     async getHello() {
       // return 'Hello World!';
       return await this.prisma.test666.findMany();
     }
   }
   ```

### 多个数据库的连接使用 [参考](https://tsegsxaviers.hashnode.dev/how-to-setup-prisma-to-use-two-different-databases-in-nestjs)

1. 先分别定义2个入口配置文件, 指定输出目录output

   ```js
   //  /prisma/schema.prisma
   generator client {
     provider = "prisma-client-js"
     output = "./generated/client/mysql"
   }
   
   datasource db {
     provider = "mysql"
     url      = env("DATABASE_URL")
   }
   
   //  /prisma/schema2.prisma
   generator client {
     provider = "prisma-client-js"
     output = "./generated/client/postgres"
   }
   
   datasource db {
     provider = "postgres"
     url      = env("DATABASE_URL2")
   }
   
   ```

2. 生成相应的客户端

   > **关于client:**  第一个客户端可以不配置output, 这样会默认生成在node_modules里, 导入时可以直接使用 import { PrismaClient } from '@prisma/client';
   >
   > **nestjs使用避坑:** 生成的客户端文件client不会被打包到dist目录里,所以会报错找不到模块,需要在`tsconfig.json`添加额外的打包目录

   ```cmd
   #  只要定义了output  必须执行以下命令
   npx prisma generate --schema prisma/schema.prisma
   npx prisma generate --schema prisma/schema2.prisma
   #  报错找不到模块 tsconfig.json文件最外层添加需要打包的额外目录
   # "include": ["src/**/*", "prisma/generated/**/*"]
   ```

3. 如果是新数据库, 直接分别执行, 这将会根据定义好的model自动在数据库生成对应的所有表

   ```cmd
   #  initname 是指定迁移文件名 迁移会同时默认执行generate命令
   npx prisma migrate dev --schema prisma/schema.prisma --initmysqlname
   npx prisma migrate dev --schema prisma/schema2.prisma --initpgname
   ```

4. 如果是已有数据库的表,执行上面命令会覆写所有数据, 所以需要先根据数据库生成模型定义,然后再执行generate命令

   ```cmd
   npx prisma db pull --schema  prisma/schema.prisma
   npx prisma db pull --schema  prisma/schema2.prisma
   ```

5. 如果未来模型定义有更新,需使用push命令

   ```cmd
   npx prisma db push --schema  prisma/schema.prisma
   npx prisma db push --schema  prisma/schema2.prisma
   ```

6. 引入和使用相应的客户端

   prisma模块module以及服务service定义

   ```ts
   // prisma.module.ts
   import { Module } from '@nestjs/common';
   import { PrismaService as MysqlService } from './prisma.service';
   import { PrismaService as PgService } from './prisma2.service';
   @Module({
     providers: [MysqlService, PgService],
     exports: [MysqlService, PgService],
   })
   export class PrismaModule {}
   
   // prisma.service.ts
   import { Injectable, OnModuleInit, OnModuleDestroy, Logger } from '@nestjs/common';
   // 如果schema.prisma没有指定output 
   import { PrismaClient } from '@prisma/client'; 
   
   @Injectable()
   export class PrismaService extends PrismaClient {}
   
   // prisma2.service.ts
   // 如果有指定output 则使用相应的client路径进行导入
   import { Injectable, OnModuleInit, OnModuleDestroy, Logger } from '@nestjs/common';
   import { PrismaClient } from '@prisma/generated/pg/client';
   
   @Injectable()
   export class PrismaService extends PrismaClient {}
   
   ```

   外部其他模块导入使用

   ```ts
   //  app.module.ts
   import { Module } from '@nestjs/common';
   import { AppController } from './app.controller';
   import { AppService } from './app.service';
   import { PrismaModule } from './prisma/prisma.module';
   @Module({
     imports: [PrismaModule],
     controllers: [AppController],
     providers: [AppService],
   })
   export class AppModule {}
   
   // app.service.ts
   import { Injectable } from '@nestjs/common';
   import { PrismaService as mysqlService } from 'src/prisma/prisma.service';
   import { PrismaService as pgService } from 'src/prisma/prisma2.service';
   @Injectable()
   export class AppService {
     constructor(
       private prisma: mysqlService,
       private prisma2: pgService,
     ) {}
     async getHello() {
       return await this.prisma.test666.findMany();
     }
     async getHello2() {
       return await this.prisma2.account_account.findMany();
     }
   }
   ```

7. **总结:**  使用prisma必须要使用`npx prisma generate生成client, 查看生成的代码会发现里面有所有表及字段的对应ts类型定义,从而实现使用过程的类型校验

8. 使用多个数据库时会存在自定义客户端存储目录, 而nestjs打包时没有编译这些文件, 从而出现找不到client的报错, 解决方案: 在`nest-cli.json`文件添加额外打包项目, 这样 `nest build` 会自动拷贝 `prisma` 目录下的文件到 `dist` 目录

   ```json
     "compilerOptions": {
       "assets": [
         {
           "include": "prisma/client/*",   // 你的自定义目录
           "watchAssets": true
         }
       ]
     }
   ```

   

   

   
