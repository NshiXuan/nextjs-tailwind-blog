---
title: 'nestjs-RBAC权限控制' # 标题
date: '2023/5/04' # 发布时间
# lastmod: '2022/3/10'
tags: [Nestjs] # 标签
draft: false
summary: 'nestjs-RBAC权限控制'
images: # 文章封面 不要就留个''空字符串
  [
    'https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ecdbb7610e3648d0885d86143fbab531~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

# nestjs-RBAC 权限控制

1.  创建表

- 如果有菜单、资源等，需要创建菜单表 `menus` 与权限的映射表 `roles_menus`

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a548833d1684322b99b95d83d0ad1da~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d06296d447fb4631869d0ac7254b8c23~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/162517f5187746d2a5aab68738ae460d~tplv-k3u1fbpfcp-zoom-1.image)

`role`

2.  创建枚举

```ts
export enum Role {
  User = 2,
  Admin = 1,
}
```

3.  创建装饰器

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b83ed1c1e565400f910a1b79bcdaff8d~tplv-k3u1fbpfcp-zoom-1.image)

- `SetMetadata` 可以将枚举 `Role` 的内容缓存，相当于 `pinio`

```ts
import { SetMetadata } from '@nestjs/common'
import { Role } from 'src/enum/roles.enum'

export const ROLES_KEY = 'roles'

// 装饰器Roles SetMetadata将装饰器的值缓存
export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles)
```

4.  创建守卫

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d02824db91934be785aeb2ea6d93273e~tplv-k3u1fbpfcp-zoom-1.image)

```
nest g gu guards/role --no-spec
```

- 从 `req` 中获取 `req.user.username` 需要配合 `nestjs` 的 `jwt` 的守卫（经过后会从 `token` 中解析出 `user` 的值放到 `req.user` 中），如下，

```ts
@UseGuards(JwtGuard, RoleGuard)
```

```ts
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common'
import { Reflector } from '@nestjs/core'
import { ROLES_KEY } from 'src/decorators/roles.decorator'
import { Role } from 'src/enum/roles.enum'
import { UserService } from 'src/user/user.service'

@Injectable()
export class RoleGuard implements CanActivate {
  constructor(private reflector: Reflector, private readonly userService: UserService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    // 1.通过反射获取到装饰器的权限
    // getAllAndOverride读取路由上的metadata getAllAndMerge合并路由上的metadata
    const requireRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ])

    // 2.获取req拿到鉴权后的用户数据
    const req = context.switchToHttp().getRequest()

    // 3.通过用户数据从数据查询权限
    const user = await this.userService.findOneByName(req.user.username)
    const roleIds = user.roles.map((item) => item.id)

    // 4.判断用户权限是否为装饰器的权限 的some返回boolean
    const flag = requireRoles.some((role) => roleIds.includes(role))

    return flag
  }
}
```

5.  在 `controller` 使用

- 方法的 `@Roles` 注解会覆盖控制器的注解

```ts
...

@Controller('roles')
@Roles(Role.Admin) // 装饰器会缓存Role.Admin的值 RoleGuard中获取缓存的值进行守卫鉴权
@UseGuards(JwtGuard, RoleGuard)
export class RolesController {
  constructor(private readonly rolesService: RolesService) {}

  ...

  @Get(':id')
  @Roles(Role.User)
  findOne(@Param('id') id: string) {
    return this.rolesService.findOne(+id)
  }

  ...
}
```
