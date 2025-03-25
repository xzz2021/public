#### nestjs后端接口权限拦截

1. 前端根据权限控制展示与否, 后端拦截接口是否允许调用. 本案例使用token存储用户角色id数组, 根据roleIds从权限表里查询出所有的权限名,从而进行拦截, 权限名采用`LOG_DELETE` `MENU_UPDATE`形式, 方便维护定位管理

2. 创建Permissions装饰器

   ```ts
   import { SetMetadata } from '@nestjs/common';
   export const Permissions = (...permissions: string[]) => SetMetadata('permissions', permissions);
   ```

3. 创建PermissionsGuard

   ```ts
   import { CanActivate, ExecutionContext, ForbiddenException, Injectable } from '@nestjs/common';
   import { Reflector } from '@nestjs/core';
   import { PrismaService as pgService } from '@/prisma/prisma.service';
   
   @Injectable()
   export class PermissionsGuard implements CanActivate {
     constructor(
       private reflector: Reflector,
       private readonly pgService: pgService,
     ) {}
     async canActivate(context: ExecutionContext): Promise<boolean> {
       const requiredPermissions = this.reflector.get<string[]>('permissions', context.getHandler());
   
       if (!requiredPermissions || requiredPermissions.length === 0) {
         return true; // 如果没有权限要求，直接通过
       }
       const request = context.switchToHttp().getRequest();
       const user = request.user; // 从JWT中获取用户信息
       if (!user) {
         throw new ForbiddenException('User not authenticated');
       }
       // 获取用户角色或权限组
       const userPermissions = await this.pgService.rolePermission.findMany({
         where: {
           roleId: { in: user.roleIds },
         },
         select: {
           permission: {
             select: {
               permissionName: true,
             },
           },
         },
       });
       const permissionNames = userPermissions.map(item => item.permission.permissionName);
       // 去重  检查用户是否拥有所有必需的权限
       const hasPermission = requiredPermissions.every(permission => [...new Set(permissionNames)].includes(permission));
   
       if (!hasPermission) {
         throw new ForbiddenException('当前用户没有操作此接口的权限');
       }
       return true;
     }
   }
   ```

4. 控制器调用

   ```ts
   @Delete('log/delete')
   @UseGuards(PermissionsGuard)  // 如果全局启用了可以注释此行
   @Permissions('LOG_DELETE')
   delete(@Body() data: { ids: number[] }) {
       return this.utilService.deleteLog(data.ids);
     }
   ```

5. 全局启用PermissionsGuard, 在入口的AppModule中配置

   ```ts
   providers: [{ provide: APP_GUARD, useClass: PermissionsGuard }]
   ```

6. 加入缓存查询功能

   ```ts
   import { Cache } from 'cache-manager';
   import { Inject } from '@nestjs/common';
   import { CACHE_MANAGER } from '@nestjs/cache-manager';
   //  ... 其余省略
   @Injectable()
   export class PermissionsGuard implements CanActivate {
     constructor(
       private reflector: Reflector,
       private readonly pgService: pgService,
       @Inject(CACHE_MANAGER) private cacheManager: Cache
     ) {}
       
    async setCache(key: string, value: string, ttl: number = 300) {
       await this.cacheManager.set(key, value, ttl);
     }
     // 获取缓存
     async getCache(key: string): Promise<string | undefined> {
       return await this.cacheManager.get(key);
     }
       // ... 逻辑同下
   }
   ```

7. 将缓存服务封装在service中全局挂载, 只需要在入口的AppModule中配置providers即可

   ```ts
   providers: [CacheService]
   ```

8. 使用cache服务获取permissionNames, 提高性能, 避免重复查询

   ```ts
     async getUserPermissions(user: { id: number; roleIds: number[] }) {
       //  使用缓存
       const cacheKey = `user_permissions_${user.id}`;
       const cachedPermissions = await this.cacheService.getCache(cacheKey);
       if (cachedPermissions) {
         return cachedPermissions;
       } else {
         const userPermissions = await this.pgService.rolePermission.findMany({
           where: {
             roleId: { in: user.roleIds },
           },
           select: {
             permission: {
               select: {
                 permissionName: true,
               },
             },
           },
         });
         const permissionNames = userPermissions.map(item => item.permission.permissionName);
         const uniquePermissionNames = [...new Set(permissionNames)];
         await this.cacheService.setCache(cacheKey, JSON.stringify(uniquePermissionNames), 300 * 1000);
         return uniquePermissionNames;
       }
     }
   // ------------------
   const userPermissions = await this.getUserPermissions(user);
   ```

   