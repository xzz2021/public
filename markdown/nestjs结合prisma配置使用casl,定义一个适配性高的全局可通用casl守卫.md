nestjs结合prisma配置使用casl,定义一个适配性高的全局可通用casl守卫

当前项目前置条件及注意事项:

> 1. 默认已配置好jwt,可以获取到用户信息; 安装依赖包 @casl/ability  @casl/prisma
> 2. @casl/prisma版本需要使用2.0.0-alpha.3(待议,正式版???可能没有条件condition能力)
> 3. 以下示例代码中PgService其实是Prisma client, 如果你是单数据库,请使用默认的@prisma/client

###### 为了全局挂载, 需要定义一个全局guard,一个权限传递decortor, 权限module及通用的factory;  CaslAbilityFactory定义有哪些能力,

###### CheckPolicies用于核对是否有当前传递的权限,未传递或管理员角色直接放行

##### 装饰器

````ts
// casl.decorator.ts
import { SetMetadata } from '@nestjs/common';
import { Action } from '@/casl/ability.factory';
import { Subjects } from '@casl/prisma';
export const CHECK_POLICIES_KEY = 'check_policy';

export interface PolicyMeta {
  action: Action;
  subject: Subjects<any>;
  conditions?: Record<string, any>;
}
export const CheckPolicies = (meta: PolicyMeta) => SetMetadata(CHECK_POLICIES_KEY, meta);
````

##### casl模块及factory

```ts
//  casl.module.ts
import { Module } from '@nestjs/common';
import { CaslAbilityFactory } from './ability.factory';

@Module({
  providers: [CaslAbilityFactory],
  exports: [CaslAbilityFactory],
})
export class CaslModule {}  

// ability.factory.ts
import { Injectable } from '@nestjs/common';
import { AbilityBuilder, PureAbility } from '@casl/ability';
//  未验证 @casl/prisma 2.0.0-alpha.3 版本 可以使用PrismaAbility  进行条件匹配
import { PrismaAbility, PrismaQuery, Subjects } from '@casl/prisma';
import { User } from '@/prisma/client/postgresql';
import { ModelInstances } from '@/prisma/casl-adapter';
export interface IUser extends User {
  roles: { code: string; id: number }[];
}
export type Action = 'manage' | 'create' | 'read' | 'update' | 'delete';
export type AppAbility = PureAbility<[string, 'all' | Subjects<ModelInstances>], PrismaQuery>;

// 作为全局使用的ability  此处可以尽可能多的定义  能力
@Injectable()
export class CaslAbilityFactory {
  createForUser(user: IUser): AppAbility {
    const { can, cannot, build } = new AbilityBuilder<AppAbility>(PrismaAbility);
    if (user.roles.some((role: { code: string }) => role.code === 'ADMIN')) {
      // can('manage', 'all');
      can('manage', 'Department');
      can('manage', 'OrderInfo');
    } else {
      // 帖子规则
      can('read', 'OrderInfo', { isDeleted: true } as any);
      // can(['update', 'delete'], 'OrderInfo', { userId: user.id });
      // cannot('delete', 'OrderInfo', { isDeleted: true });
      // 部门规则
      can('read', 'Department');
      // 普通用户只能更新 status 为 true 的部门
      can('update', 'Department');
      cannot('update', 'Department', { status: false });
      // （普通用户无法增删改部门）
    }
    return build();
  }
}
```

casl-adapter是由generate自动生成的,需要在schema文件定义,然后自己提取ModelInstances

```ts
//  casl-adapter
import { Prisma } from './client/postgresql';

export type WhereInput<T extends Prisma.ModelName> = {
  User: Prisma.UserWhereInput;
  WechatInfo: Prisma.WechatInfoWhereInput;
  Role: Prisma.RoleWhereInput;
  Department: Prisma.DepartmentWhereInput;
  Menu: Prisma.MenuWhereInput;
  Permission: Prisma.PermissionWhereInput;
  Notice: Prisma.NoticeWhereInput;
  NoticeRecipient: Prisma.NoticeRecipientWhereInput;
  Dictionary: Prisma.DictionaryWhereInput;
  DicEntry: Prisma.DicEntryWhereInput;

}[T];
export type ModelName = Prisma.ModelName;

export type ModelInstances = {
  User: Prisma.UserWhereInput;
  WechatInfo: Prisma.WechatInfoWhereInput;
  Role: Prisma.RoleWhereInput;
  Department: Prisma.DepartmentWhereInput;
  Menu: Prisma.MenuWhereInput;
  Permission: Prisma.PermissionWhereInput;
  Notice: Prisma.NoticeWhereInput;
  NoticeRecipient: Prisma.NoticeRecipientWhereInput;
  Dictionary: Prisma.DictionaryWhereInput;
  DicEntry: Prisma.DicEntryWhereInput;

};

```

schema文件添加以下生成代码

```ts
generator caslAdapter {
  provider = "node node_modules/@casl/prisma/generator.js" // <--- important to add this
  clientLib = "./client/postgresql" // optional and by default equals to @prisma/client
  output  = "./casl-adapter.ts"
}
```

#####  casl守卫

````ts
// casl.guard.ts
//  此处条件查询判断的 实体来源 是根据请求头数据中的id解析  默认id就是当前操作的数据
//  后期 应当考虑数组 批量处理的情况
import { Injectable, CanActivate, ExecutionContext, ForbiddenException } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { CHECK_POLICIES_KEY, PolicyMeta } from '@/processor/decorator/casl.decorator';
import { CaslAbilityFactory, IUser } from '@/casl/ability.factory';
import { IS_PUBLIC_KEY } from '../decorator/public.decorator';
import { PgService } from '@/prisma/pg.service';
import { subject } from '@casl/ability';

@Injectable()
export class PoliciesGuard implements CanActivate {
  constructor(
    private reflector: Reflector,
    private caslFactory: CaslAbilityFactory,
    private pgService: PgService,
  ) {}

  async canActivate(ctx: ExecutionContext): Promise<boolean> {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [ctx.getHandler(), ctx.getClass()]);
    if (isPublic) return true;

    const handler = this.reflector.get<PolicyMeta>(CHECK_POLICIES_KEY, ctx.getHandler());
    if (!handler) return true;
    const req = ctx.switchToHttp().getRequest<Request & { user: IUser }>();
    const user: IUser = req.user;
    if (!user) {
      throw new ForbiddenException('请先登录');
    }

    if (user.roles.some(role => role.code === 'ADMIN')) {
      return true; // 管理员直接放行
    }
    const ability = this.caslFactory.createForUser(user);
    const { action, subject: subjectName } = handler;

    let newSubject = subjectName;
    const operateId = this.extractOperateId(req);
    if (operateId) {
      const currentInstance = await this.pgService[subjectName.toLowerCase()].findFirst({
        where: { id: operateId },
      });
      // if (!currentInstance) return false;
      if (!currentInstance) {
        throw new ForbiddenException('数据库资源不存在, 检查实体subject参数或id是否存在!');
      }
      newSubject = currentInstance;
    }

    // eslint-disable-next-line @typescript-eslint/no-unsafe-argument
    const allowed = ability.can(action, typeof newSubject === 'string' ? newSubject : subject(subjectName, newSubject));

    if (!allowed) {
      throw new ForbiddenException('没有权限操作此资源');
    }
    return true;
  }

  private extractOperateId(request: any): number | undefined {
    // 优先级：params > body > query（如有需要）
    return Number(request.params?.id) || Number(request.body?.id) || Number(request.query?.id) || undefined;
  }
}

//   后期  可扩展为多个策略支持

````

##### 全局挂载守卫

````ts
 [
  {
   // 全局JWT token校验
    provide: APP_GUARD,
    useClass: JwtAuthGuard,
  },

  //  全局casl权限校验
  {
    provide: APP_GUARD,
    useClass: PoliciesGuard,
  }
   ]
````

##### 使用示例

```ts
  @Post('update')
  @CheckPolicies({ action: 'update', subject: 'Department' })
  update(@Body() updateDepartmentDto: UpdateDepartmentDto) {
    return this.departmentService.update(updateDepartmentDto);
  }
```



