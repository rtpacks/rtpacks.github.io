---
title: 记录一次 DTO，Pipe，class-validator 结合使用过程中 class-validator 不生效的原因
date: 2023-03-19 22:53:18
categories: NestJS
tags: 
- NestJS
---

记录一次 DTO，Pipe，class-validator 结合使用过程中 class-validator 不生效的原因

### 解决方法

1. 自定义 transform 方法中的 value 是一个来自前端的 `json obj`，需要将其转换成你用 `class-validator` 的装饰器修饰的 `class`，如 class UserDTO 或 class User（用 @IsNotEmpty 装饰器修饰字段/类的 class），**只有转换后校验才会生效！** 转换方法：

```ts
@Injectable()
export class TransformUserPipe implements PipeTransform {
  async transform(value: any) {
    // 将来自请求的数据进项转换
    value = plainToClass(UserDTO, value);
  }
}
```

2. 使用 `validate` 方法校验数据

```ts
@Injectable()
export class TransformUserPipe implements PipeTransform {
  async transform(value: any) {
    // 将来自请求的数据进项转换
    value = plainToClass(UserDTO, value);
    const errors = await validate(value);
    if (errors.length) {
      throw new BadRequestException('参数错误');
    }
  }
}
```

3. 如果不生效，检查一下是否加入了全局 或 Control 或 Method 或 参数

```ts
@Post()
@UsePipes(new TransformUserPipe())
async postUser(@Body() user: User): Promise<User> {
  return await this.userService.create(user);
}

@Post()
async postUser(@Body(new TransformUserPipe()) user: User): Promise<User> {
  return await this.userService.create(user);
}
```

4. 检查一下是否开启了 ValidationPipe

最好在 main.ts 中开启一下 `app.useGlobalPipes(new ValidationPipe());`，如果你能保证在每一个接口都能保持需要的数据类型，那么开不开没多大的意义。

### 详细介绍

因为刚上手 NestJS，对这个不太属性，刚开始我是直接将 DTO 对象(class) 拿到了 Control 层，这也没什么事情能够获取到数据，但是 Control 的各种判断和转换实在难看了，就想着有没有转换管道 Pipe，在查看文档之后发现是有的。

在查看了一番文档之后，便开始动手改造我的 DTO 和 Control

首先是 DTO，为了不写 if-else，我选择使用 class-validator 这个库，这个库具有非常丰富的校验装饰器，具体的可以看看 [Nest class-validator 验证修饰器中文文档](https://blog.csdn.net/qq_38734862/article/details/117265394)

修改完成之后，我的 DTO 为：

```ts
export class UserDTO {
  /**
   * uuid
   */
  @IsNotEmpty({ message: '用户ID必须存在' })
  @IsUUID(undefined, { message: '用户ID必须是UUID形式' })
  id: string;

  /**
   * 用户名称
   */
  @IsOptional()
  @IsString({ message: '用户名必须是字符串！' })
  @Length(6, 20, {
    message: '用户名长度必须为 $constraint1 到 $constraint2 之间',
  })
  username?: string;

  /**
   * 用户密码
   */
  @IsOptional()
  @IsString({ message: '密码必须是字符串！' })
  @Length(6, 20, {
    message: '密码长度必须为 $constraint1 到 $constraint2 之间',
  })
  password?: string;

  // ...
}
```

其中，ID 是必须存在的，其他可以为不传。顺便提一句，如果更新的字段值为 undefined，TypeOrm 是不会进行更新的，只有设置空（null）才会更新。

其次是 Pipe：

```ts
@Injectable()
export class TransformUserPipe implements PipeTransform {
  // 如果参数名称和原有的一样，那么可以直接使用 ClassTransformerPipe
  async transform(u: Partial<UserDTO>): Promise<User> {
    const user = new User();
    user.id = u.id;
    user.username = u.username;
    user.password = u.password;
    return user;
  }
}
```

感觉很奇怪，这种怎么会生效，再次查找之后，是需要加上 validate 进行验证，结果 class-validator 还是不生效

```ts
@Injectable()
export class TransformUserPipe implements PipeTransform {
  // 如果参数名称和原有的一样，那么可以直接使用 ClassTransformerPipe
  async transform(u: Partial<UserDTO>): Promise<User> {
    // 开始校验
    const errors = await validate(u);
    if (errors.length) {
      throw new BadRequestException('参数错误');
    }

    const user = new User();
    user.id = u.id;
    user.username = u.username;
    user.password = u.password;
    return user;
  }
}
```

再次查找之后，发现这个 u 很奇怪，按照道理这阶段是 Pipe，接受的数据应该是前端发过来的 json 数据，是我想当然的将它当成 Partial\<UserDTO\>了，它并不是一个 `class`，而是一个 `json obj`，所以应该使用 Nest.js 提供的 plainToClass 函数进行转换，最后结合校验，完成功能

```ts
@Injectable()
export class TransformUserPipe implements PipeTransform {
  async transform(u: Partial<UserDTO>): Promise<User> {
    // 将来自请求的数据进项转换
    u = plainToClass(UserDTO, u);

    // 开始校验转换类型后的数据
    const errors = await validate(u);
    if (errors.length) {
      const errMsg = errors
        .map((e) => {
          const constrains = e.constraints;
          return Object.values(constrains).join('; \n');
        })
        // 为避免返回过多数据，限制10条
        .filter((_, i) => i < 10)
        // 再次格式化换行
        .join('; \n');
      // log
      throw new BadRequestException(errMsg);
    }

    const user = new User();
    user.id = u.id;
    user.username = u.username;
    user.password = u.password;
    return user;
  }
}
```

最后，最好在 main.ts 中开启一下 `app.useGlobalPipes(new ValidationPipe());`，如果你能保证在每一个接口都能保持需要的数据类型，那么开不开没多大的意义。

### 其他

- [记录一次 DTO，Pipe，class-validator 结合使用过程中 class-validator 不生效的原因 | class-validator 不生效_一抹阳光&的博客-CSDN博客](https://blog.csdn.net/qq_45759413/article/details/129660140)
- [记录一次 DTO，Pipe，class-validator 结合使用过程中 class-validator 不生效的原因 | Az's Blog (azin-cn.github.io)](https://azin-cn.github.io/2023/03/19/记录一次-DTO，Pipe，class-validator-结合使用过程中-class-validator-不生效的原因/)
