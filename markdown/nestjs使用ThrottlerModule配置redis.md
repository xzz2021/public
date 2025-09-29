nestjs使用ThrottlerModule配置redis

需求: 全局接口根据不同用户限流(默认的是根据接口,无论谁请求)

1. 先配置ThrottlerModule,本项目使用的是`@liaoliaots/nestjs-redis`, 结合ioredis, 所以需要`ThrottlerStorageRedisService`包裹, 也可以直接配置url,不需要此依赖

   ```ts
   import { RedisService } from '@liaoliaots/nestjs-redis';
   import { GlobalThrottlerGuard } from '@/processor/guard/global-throttler.guard';
   import { ThrottlerModule } from '@nestjs/throttler';
   import { ThrottlerStorageRedisService } from '@nest-lab/throttler-storage-redis';
   
   ThrottlerModule.forRootAsync({
       inject: [RedisService],
       useFactory: (redisService: RedisService) => ({
         // 新版  storage  需要写在顶层
         storage: new ThrottlerStorageRedisService(redisService.getOrThrow('default')),
         throttlers: [
           {
             ttl: 60 * 1000,
             limit: 100,
           },
         ],
       }),
     })
   ```

2. 继承官方ThrottlerGuard, 实现自定义guard, 核心是根据用户身份自定义key名

   ```ts
   // custom-throttler.guard.ts
   import { Injectable, ExecutionContext } from '@nestjs/common';
   import { ThrottlerGuard } from '@nestjs/throttler';
   import { RATE_KEY } from '../decorator/rate-key.decorator';
   
   @Injectable()
   export class GlobalThrottlerGuard extends ThrottlerGuard {
     // eslint-disable-next-line @typescript-eslint/require-await
     protected async getTracker(req: any, context?: ExecutionContext): Promise<string> {
       // 1. 优先读取装饰器指定的 key
       if (context) {
         const handlerKey = this.reflector.get<string>(RATE_KEY, context.getHandler());
         if (handlerKey) {
           return `custom:${handlerKey}`;
         }
       }
       // 2. 登录用户
       if (req.user?.id) return `u:${req.user.id}`;
       // 3. API Key
       const apiKey = req.headers['x-api-key'] as string;
       if (apiKey) return `k:${apiKey}`;
       // 4. 未登录：回退到 IP + UA，降低共享IP误伤
       const xff = (req.headers['x-forwarded-for'] as string) || '';
       const ip = (xff.split(',')[0] || '').trim() || req.ip || req.connection?.remoteAddress || 'unknown';
       const ua = req.headers['user-agent'] || '';
       return `ip:${ip}|ua:${ua}`;
     }
   }
   ```

3. 全局守卫

   ```ts
     {
       // 重写自定义全局 ThrottlerGuard  的 key名  从而实现根据不同用户进行限流
       provide: APP_GUARD,
       useClass: GlobalThrottlerGuard,
     },
   ```

4. 如果需要对单独接口进行限流配置, 可以单独使用Throttle装饰器

   ```ts
   @Throttle({ default: { limit: 3, ttl: 50000 } })
   ```

5. 部分原理剖析

   * 官方key生成逻辑: key ≈ `sha256( tracker + ':' + className + ':' + handlerName + ':' + throttlerName )`, 最终存入redis的key都是一个hash值
   * 所以重新实现的getTracker只是最终生成的key的一部分,如果不自定义实现,所有人请求的tracker都是一样的