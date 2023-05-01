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

## 增删改查

1. 在 `user.module.ts` 中导入 `TypeOrmModule`

```ts
import { Module } from '@nestjs/common'
import { UserService } from './user.service'
import { UserController } from './user.controller'
import { User } from './entities/user.entity'
import { TypeOrmModule } from '@nestjs/typeorm'

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UserController],
  providers: [UserService],
})
export class UserModule {}
```

2. `dto`

```ts
export class CreateUserDto {
  username: string
  password: string
}
```

3. `user.service.ts`

```ts
import { Injectable } from '@nestjs/common'
import { CreateUserDto } from './dto/create-user.dto'
import { UpdateUserDto } from './dto/update-user.dto'
import { InjectRepository } from '@nestjs/typeorm'
import { User } from './entities/user.entity'
import { Repository } from 'typeorm'

@Injectable()
export class UserService {
  constructor(@InjectRepository(User) private userRepository: Repository<User>) {}

  create(createUserDto: CreateUserDto) {
    return this.userRepository.save(createUserDto)
  }

  findAll() {
    return this.userRepository.find()
  }

  findOne(id: number) {
    return this.userRepository.findOne({ where: { id } })
  }

  async update(id: number, updateUserDto: Partial<User>) {
    await this.userRepository.update(id, updateUserDto)
    return this.findOne(id)
  }

  remove(id: number) {
    return this.userRepository.delete(id)
  }
}
```

4. `controller`

```ts
import { Controller, Get, Post, Body, Patch, Param, Delete } from '@nestjs/common'
import { UserService } from './user.service'
import { UpdateUserDto } from './dto/update-user.dto'
import { User } from './entities/user.entity'
import { CreateUserDto } from './dto/create-user.dto'

@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto)
  }

  @Get()
  findAll() {
    return this.userService.findAll()
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.userService.findOne(+id)
  }

  @Patch(':id')
  update(@Param('id') id: string, @Body() updateUserDto: Partial<User>) {
    return this.userService.update(+id, updateUserDto)
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.userService.remove(+id)
  }
}
```

## 联合关系

### OneToOne

1. 定义 `entities`

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
import { Column, Entity, OneToOne, PrimaryGeneratedColumn } from 'typeorm'
import { Profile } from './profile.entity'

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number

  @Column()
  username: string

  @Column()
  password: string

  @OneToOne(() => Profile, (profile) => profile.user)
  profile: Profile
}
```

2. 查询

```ts
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User) private userRepository: Repository<User>
  ) { }

  ...

  findProfile(id: number) {
    return this.userRepository.find({
      where: {
        id
      },
      relations: {
        profile: true
      }
    })
  }
}
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f927d73b1fa4d5abe7204ce13c831c9~tplv-k3u1fbpfcp-watermark.image?)

### OneToMany 与 ManyToMany

1. 定义 `entities`

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

2. 查询

```ts
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User) private userRepository: Repository<User>,
    @InjectRepository(Logs) private logsRepository: Repository<Logs>
  ) { }

  ...

  async findUserLogs(id: number) {
    const user = await this.findOne(id)
    return this.logsRepository.find({
      where: {
        user
      }
      // relations: {
      //   user: true
      // }
    })
  }
}
```

注释 `relations` ，不会把 `user` 查出来

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/250e76bbc1644252b9f4cc6824b1b89d~tplv-k3u1fbpfcp-watermark.image?)

使用 `relations` ，把 `user` 查出来

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5cee62c4103d46838b2b045c390d5556~tplv-k3u1fbpfcp-watermark.image?)

## QueryBuilder

### 基本使用

`service`

```ts
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(Logs) private logsRepository: Repository<Logs>
  ) {}

  ...

  getLogsByGroup(id: number): Promise<any[]> {
    // SELECT logs.result as result, COUNT(logs.result) as count from logs, user WHERE user.id = logs.userId AND user.id = 2 GROUP BY logs.result

    // logs为别名
    return this.logsRepository
      .createQueryBuilder('logs')
      .select('logs.result', 'result')
      .addSelect('COUNT("logs.result")', 'count') // count为COUNT("logs.result")的别名
      .leftJoinAndSelect('logs.user', 'user')
      .where('user.id=:id', { id })
      .groupBy('logs.result')
      .orderBy('result', 'DESC') // 根据result倒序排列
      .addOrderBy('count', 'DESC') // 如果result相同 根据count倒序排列
      .limit(3) // 只查3条
      .getRawMany()
  }
}
```

`controller`

```ts
@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get('/logsByGroup/:id')
  async getLogsByGroup(@Param('id') id: string) {
    const res = await this.userService.getLogsByGroup(+id)

    // 映射返回数据 只返回 result 与 count
    return res.map((item) => ({
      result: item.result,
      count: item.count,
    }))
  }
}
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/474bed4ce71243d0ab846fd3ac019b50~tplv-k3u1fbpfcp-watermark.image?)

### 原生使用

`service`

```ts
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User) private userRepository: Repository<User>,
    @InjectRepository(Logs) private logsRepository: Repository<Logs>
  ) {}

  getLogsByGroup(id: number): Promise<any[]> {
    return this.logsRepository.query(
      'SELECT logs.result as result, COUNT(logs.result) as count from logs, user WHERE user.id = logs.userId AND user.id = 2 GROUP BY logs.result'
    )
  }
}
```
