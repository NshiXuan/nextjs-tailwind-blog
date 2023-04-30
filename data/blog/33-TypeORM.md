---
title: 'TypeORM' # 标题
date: '2023/4/30' # 发布时间
# lastmod: '2022/3/10'
tags: [Nestjs] # 标签
draft: false
summary: 'TypeORM'
images: # 文章封面 不要就留个''空字符串
  [
    'https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ecdbb7610e3648d0885d86143fbab531~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

## 安装 typeorm

```
pnpm i -save @nestjs/typeorm typeorm mysql2
```

## 在 nestjs 中基本使用

1.  创建 `env` 配置环境与枚举常量

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9942a4b0059e4e36a3d527aefc21d9e3~tplv-k3u1fbpfcp-watermark.image?)

```
NODE_ENV=development

DB_TYPE=mysql
DB_HOST=localhost
DB_PORT=3307

DB_DATABASE=testdb
DB_USERNAME=root
DB_PASSWORD=example

DB_SYNC=false
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/298466f90c5c4e84a436d603cf0a833a~tplv-k3u1fbpfcp-watermark.image?)

```ts
export enum ConfigEnum {
  DB_TYPE = 'DB_TYPE',
  DB_HOST = 'DB_HOST',
  DB_PORT = 'DB_PORT',
  DB_DATABASE = 'DB_DATABASE',
  DB_USERNAME = 'DB_USERNAME',
  DB_PASSWORD = 'DB_PASSWORD',
  DB_SYNC = 'DB_SYNC',
}
```

2. 创建 `entity`

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/472c6e29b2304e4cb3baf95b9ba3cd5a~tplv-k3u1fbpfcp-watermark.image?)

```ts
import { Logs } from 'src/logs/logs.entity'
import { Roles } from 'src/roles/entity/roles.entity'
import { Column, Entity, PrimaryGeneratedColumn } from 'typeorm'

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number

  @Column()
  username: string

  @Column()
  password: string
}
```

3.  在 `app.module` 引入

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30b756e678694982bf995fb1e83ed806~tplv-k3u1fbpfcp-watermark.image?)

- 在 `TypeOrmModule` 的 `entities` 导入上面定义 `User`，再使用 `synchronize` 初始化数据表
- `synchronize` 为 `true` 表示将自动同步实体和数据库（如果数据库中不存在相应表），而 `false`表示不进行自动同步，即不执行数据库的自动迁移。

```ts
/* eslint-disable prettier/prettier */
import { Module } from '@nestjs/common'
import { ConfigModule, ConfigService } from '@nestjs/config'
import { AppController } from './app.controller'
import { AppService } from './app.service'
import Configuration from './configuration'
import { TypeOrmModule, TypeOrmModuleOptions } from '@nestjs/typeorm'

import { ConfigEnum } from './enum/config.enum'
import { UserModule } from './user/user.module'
import { User } from './user/entities/user.entity'
import { Profile } from './user/entities/profile.entity'
import { Roles } from './roles/entity/roles.entity'
import { Logs } from './logs/logs.entity'

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true, // 定义为全局
    }),
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) =>
        ({
          type: config.get(ConfigEnum.DB_TYPE),
          host: config.get(ConfigEnum.DB_HOST),
          port: config.get(ConfigEnum.DB_PORT),
          username: config.get(ConfigEnum.DB_USERNAME),
          password: config.get(ConfigEnum.DB_PASSWORD),
          database: config.get(ConfigEnum.DB_DATABASE),
          entities: [User],
          // 同步本地的schema与数据库 -> 初始化的时候使用
          synchronize: config.get(ConfigEnum.DB_SYNC),
          // 设置日志等级
          logging: ['error'],
        } as TypeOrmModuleOptions),
    }),
    UserModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

4. 结果

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25bab41c114a4ff79808cad3d44738d5~tplv-k3u1fbpfcp-watermark.image?)

## 联合关系

### OneToOne

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84aead0732c34cb597e9774b9fe379f5~tplv-k3u1fbpfcp-watermark.image?)

```ts
import { Column, Entity, JoinColumn, OneToOne, PrimaryGeneratedColumn } from 'typeorm'
import { User } from './user.entity'

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number

  @Column()
  gender: number

  @Column()
  photo: string

  @Column()
  address: string

  // 一对一关联关系
  @OneToOne(() => User)
  // 默认加入的关联关系字段名会以表名与主键小驼峰连接 userId
  @JoinColumn()
  // name自定义关联关系的字段名称
  // @JoinColumn({ name: 'user_id' })
  // 查询(也就是join)时会查userId与User表的id相同的数据 并将数据塞入user
  user: User
}
```

`user`

```ts
import { Logs } from 'src/logs/logs.entity'
import { Roles } from 'src/roles/entity/roles.entity'
import { Column, Entity, JoinTable, ManyToMany, OneToMany, PrimaryGeneratedColumn } from 'typeorm'

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number

  @Column()
  username: string

  @Column()
  password: string
}
```

### OneToMany 与 ManyToMany

`User`

```ts
import { Logs } from 'src/logs/logs.entity'
import { Roles } from 'src/roles/entity/roles.entity'
import { Column, Entity, JoinTable, ManyToMany, OneToMany, PrimaryGeneratedColumn } from 'typeorm'

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number

  @Column()
  username: string

  @Column()
  password: string

  // oneToMany 一个用户有多条log日志输出
  // logs.user代表的是logs表中的userId 告诉服务器去查询user表的id与logs表中的userId相等的数据 就是join
  // 然后把查询到的数据塞给logs
  @OneToMany(() => Logs, (logs) => logs.user)
  logs: Logs[]

  @ManyToMany(() => Roles, (roles) => roles.users)
  // 生成多对多关联表
  @JoinTable({ name: 'users-roles' })
  roles: Roles[]
}
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41167e0cc8d944779aadb93a739b75f9~tplv-k3u1fbpfcp-watermark.image?)

`Logs`

```ts
import { User } from 'src/user/entities/user.entity'
import { Column, Entity, JoinColumn, ManyToOne, PrimaryGeneratedColumn } from 'typeorm'

@Entity()
export class Logs {
  @PrimaryGeneratedColumn()
  id: number

  @Column()
  path: string

  @Column()
  method: string

  @Column()
  data: string

  @Column()
  result: string

  // ManyToOne 多条日志是同一个用户输出
  @ManyToOne(() => User, (user) => user.logs)
  // 生成userId字段
  @JoinColumn()
  user: User
}
```

`Roles`

```ts
import { User } from 'src/user/entities/user.entity'
import { Column, Entity, ManyToMany, PrimaryGeneratedColumn } from 'typeorm'

@Entity()
export class Roles {
  @PrimaryGeneratedColumn()
  id: number

  @Column()
  name: string

  @ManyToMany(() => User, (user) => user.roles)
  users: User[]
}
```

## 数据库生成实体类

1. 安装生成器依赖

```
pnpm i typeorm-model-generator -D
```

2.配置 `package.json` 命令

- `-p` 端口
- `-d` 数据库
- `-x` 后接密码
- `-o` 后接生成实体类的存放路径

```json
"generate:models": "typeorm-model-generator -h localhost -p 3307 -d testdb -u root -x example -e mssql -o ./src/entities"
```

3. 运行命令

```
pnpm generate:models
```
