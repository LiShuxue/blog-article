## Nest 是什么

### 介绍

[中文文档](https://docs.nestjs.cn/10/introduction)

Nest 是一个基于 TypeScript 的 Node.js 框架，它使用了 OOP（面向对象编程）、 AOP（面向切面编程）、 DI（依赖注入）等思想。

Nest 提供了一系列强大的组件，包括模块、控制器、服务、提供者、中间件、过滤器、管道、拦截器和守卫等，使您能够构建可维护、可扩展的企业级应用程序。

除了 Nest 这个框架，你可能也听说过像 Koa、Express、Egg 等这样的框架。其实 Node.js 框架也是为了更好地组织和封装代码，就像 Vue 或者 React 对于 JavaScript，或者 Spring 对于 Java 一样。

### 前端圈都在学 Nest

今年年初那会儿，业内流传出一句话，“前端已死”。前端死了怎么办呢，那我们就开始卷后端，成为全栈工程师。

作为前端工程师，学习 Nodejs 无疑是了解后端最快的途径，所以学习框架 Nest 也是目前一个很好的选择。

Nest 也是最像 spring boot 的框架，所以了解了这个对以后学习 spring 都很有帮助。

### 为什么选择 Nest

NestJS 提供了许多优势，使它成为构建 Node.js 应用的理想选择：

1. 完整的框架： NestJS 提供了一个完整的框架，内置了路由处理、依赖注入、中间件、异常处理等功能，简化了开发流程。

2. TypeScript 支持： TypeScript 让您在开发过程中拥有强类型检查和更好的代码组织，提供了更好的开发体验和代码可维护性。

3. 模块化架构： NestJS 鼓励使用模块化的方式组织代码，将应用程序拆分为各个功能模块，有助于保持代码的清晰性和可扩展性。

4. 依赖注入： 通过依赖注入，您可以更好地管理组件之间的关系，提高代码的可测试性和可维护性。

5. 强大的中间件支持： NestJS 内置了许多中间件，例如日志记录、认证、身份验证等，简化了常见任务的处理。

## Nest 的核心功能

### 模块（Modules）

模块是代码组织的基本单元。每个应用程序都由一个或多个模块组成，至少有一个根模块。模块负责将应用程序的不同部分组合在一起。

通过 @Module 装饰器来定义一个模块。@Global 装饰器可以使模块成为全局作用域。

当启动 NestJS 应用程序时，它会首先加载主模块（通常是根模块），然后递归加载所有与主模块关联的模块，依次初始化它们。

```ts
import { Module } from '@nestjs/common';
import { CatController } from './cat.controller';
import { CatService } from './cat.service';

@Module({
  imports: [], // 当前模块所依赖的其他模块
  controllers: [CatController], // 当前模块的控制器
  providers: [CatService], // 当前模块中提供的服务、提供者和其他可注入的对象
  exports: [], // 将当前模块的某个组件导出，其他模块才可以导入当前模块中导出的组件。
})
export class CatModule {}
```

imports 字段定义了当前模块所依赖的其他模块。导入其他模块的功能意味着您可以访问这些模块中的控制器、服务、提供者等。

controllers 字段用于指定当前模块中包含的控制器。

providers 字段定义了当前模块中提供的服务、提供者和其他可注入的对象。提供者可以是服务、工厂、库等。

exports 字段用于将当前模块的功能导出到其他模块中。通过将模块的功能导出，其他模块可以使用当前模块中定义的控制器、服务等。

### 控制器（Controllers）

控制器是处理应用程序 HTTP 请求的组件，它们是应用程序的入口点。

每个控制器负责处理特定路由下的请求，并将请求逻辑委托给相应的服务。

```ts
import { Controller, Get } from '@nestjs/common';
import { CatService } from './cat.service';

@Controller('cat')
export class CatController {
  // 当控制器被实例化的时候，CatService会被注入，自动实例化catService（默认是单例的，如果已经存在，直接返回）
  constructor(private readonly catService: CatService) {}

  @Get()
  findAll() {
    console.log('controller');
    const catList = this.catService.findAll();
    return catList;
  }
}
```

使用 @Controller('cat') 注解声明一个 Controller，处理 cat 路径下的请求。

使用 @Get @Put @Post @Delete 来定义请求类型和路径。使用@Param @Body @Query @Headers 等来获取对应 request 中的信息。

@Query() 用于从查询字符串?xxx=yyy 中提取参数值，而 @Param() 用于从路由参数/user/:id 中提取参数值。

### 服务（Services）

通过 @Injectable() 装饰器可以声明一个服务类，服务是处理应用程序业务逻辑的组件。它们处理从控制器传递的数据，并执行特定的操作。通过依赖注入的方式在控制器中进行实例化和使用。

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class CatService {
  findAll() {
    console.log('service');
    // 这里可以调用数据库方法去查询数据库并返回
    return [];
  }
}
```

### 提供者（Providers）

通过将 @Injectable() 装饰器应用于类，标识这个类是可注入的，可注入的类就是提供者。

提供者会被声明在 @Module 的 providers 中，将被依赖注入到 Controller 或者其他地方。

### 中间件（Middlewares）

中间件是在路由处理程序 **之前** 调用的函数。可以访问请求和响应对象，以及请求响应周期中的 next() 函数。可以用来处理请求、进行验证、日志记录、实现认证等操作。

中间件可以使用类的形式，也可以是函数形式。

```ts
// 1. 函数形式
export function logger(req, res, next) {
  console.log('middleware');
  next();
}

// 2. 使用类的形式声明中间件
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('middleware');
    // 执行下一个中间件或路由处理程序
    next();
  }
}
```

中间件也是用 @Injectable() 声明，但是中间件不能在 @Module() 装饰器中列出。需要使用模块类的 configure() 方法来设置它们，所以包含中间件的模块必须实现 NestModule 接口，实现 configure 方法。

apply 方法来使用中间件，可以使用多个。可以使用在特定的路径 forRoutes('cat')、特定的控制器 forRoutes(CatController)，或者是所有路径 forRoutes('\*')中。exclude() 方法可以排除某些路由。

```ts
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { logger } from './logger';
import { CatModule } from './moudules/cat/cat.module';

@Module({
  imports: [CatModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(logger) // 可以使用多个中间件
      .forRoutes('cat'); // 可以使用在特定的路径forRoutes('cats')，或者特定的控制器forRoutes(CatsController)，或者是所有路径forRoutes('*')
  }
}
```

### 拦截器（Interceptors）

拦截器用于 在请求到达控制器之前 或 响应返回客户端之前 添加额外的逻辑。它们可以用于修改请求或响应对象。

拦截器也是使用 @Injectable() 装饰器的类。拦截器应该实现 NestInterceptor 接口，并且实现 intercept 方法。

next.handle() 返回一个 Observable，此流包含从路由处理程序返回的值（响应）, 我们可以使用 map() 运算符地对其进行改变。

```ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class MyInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('interceptor Before request...');
    return next.handle().pipe(
      // next.handle() 返回一个 Observable，此流包含从路由处理程序返回的值（响应）, 我们可以使用 map() 运算符地对其进行改变。
      map((data) => {
        console.log('interceptor After response...');
        // 在这里对响应数据进行转换或操作
        return data; // 返回转换后的数据
      })
    );
  }
}
```

@UseInterceptors 可以将拦截器使用在局部方法上，参数是 class，可以依赖注入，参数是 instance，则不可以注入。

也可以通过 app.useGlobalInterceptors 使用在全局。可以传多个拦截器，按顺序处理。

```ts
// 局部
@Controller('cat')
@UseInterceptors(MyInterceptor) // 自动依赖注入
@UseInterceptors(new MyInterceptor()) // 不会依赖注入
export class CatController {
  ...
}

// 全局
const app = await NestFactory.create(ApplicationModule);
app.useGlobalInterceptors(new MyInterceptor());
```

这种全局方法，没有办法进行依赖注入，需要 new XXX() 中传参。如果想在 constructor 中依赖注入其他 service，需要在 app.module 中的 providers 中声明。

```ts
providers: [
  {
    provide: APP_INTERCEPTOR,
    useClass: HttpInterceptor, // 全局注册您的拦截器
  },
],
```

### 过滤器（Filters）

过滤器（Filters）是用于处理异常的组件，它们允许您在应用程序中统一处理抛出的异常，以提供更友好和一致的错误响应。通常是使用 @Catch() 装饰器进行注解的类。

能够在全局范围或特定控制器范围内捕获异常，并根据异常类型提供自定义的响应。

它是响应返回给客户端的最后一次过滤。

```ts
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    console.log('HttpExceptionFilter');

    // 在这里处理异常并返回响应
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}
```

过滤器可以通过 @UseFilters 使用在某个路径上，参数是 class，可以依赖注入，参数是 instance，则不可以注入。

也可以 app.useGlobalFilters 用在全局。可以传入多个过滤器。

```ts
//局部
@Controller('cat')
@UseFilters(HttpExceptionFilter) // 自动依赖注入
@UseFilters(new HttpExceptionFilter()) // 不会自动注入
export class CatController {
  ...
}

// 全局
const app = await NestFactory.create(AppModule);
app.useGlobalFilters(new HttpExceptionFilter());
```

这种全局方法，没有办法进行依赖注入，需要 new XXX() 中传参。如果想在 constructor 中依赖注入其他 service，需要在 app.module 中的 providers 中声明。

```ts
providers: [
  {
    provide: APP_FILTER,
    useClass: HttpExceptionFilter, // 全局注册您的过滤器
  },
],
```

### 守卫（Guards）

守卫在每个中间件之后执行，但在任何拦截器或管道之前执行。一般用来做鉴权，来确定请求是否可以继续。

守卫也是一个使用 @Injectable() 装饰器的类。守卫应该实现 CanActivate 接口，且必须实现一个 canActivate() 函数，此函数应该返回一个布尔值，用于指示是否允许当前请求。

```ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    console.log('guard');
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}

const validateRequest = (req) => {
  return true;
};
```

守卫也是可以控制使用范围的，方法范围 @UseGuards() 参数是 class，可以依赖注入，参数是 instance，则不可以注入。

全局范围 app.useGlobalGuards。

```ts
// 方法范围
@Controller('cat')
@UseGuards(AuthGuard)
@UseGuards(new AuthGuard())
export class CatController {}

// 全局范围
const app = await NestFactory.create(AppModule);
app.useGlobalGuards(new AuthGuard());
```

这种全局方法，没有办法进行依赖注入，需要 new XXX() 中传参。如果想在 constructor 中依赖注入其他 service，需要在 app.module 中的 providers 中声明。

```ts
providers: [
  {
    provide: APP_GUARD,
    useClass: AuthGuard,
  },
],
```

### 管道（Pipes）

管道一般用于控制器的路由处理程序中，用于在数据传递给控制器之前对其进行验证、转换和处理。管道要么返回数据验证或者转换后的值，要么抛出一个错误。

管道也是具有 @Injectable() 装饰器的类。管道应实现 PipeTransform 接口，且实现 transform 方法。

当 class-validator 结合 dto 的验证不满足需求时，才需要自定义 pipe。

```ts
import { Injectable, PipeTransform, ArgumentMetadata } from '@nestjs/common';

@Injectable()
export class ValidationPipe implements PipeTransform {
  // metadata 参数是一个包含了有关参数的元数据的对象，其中包括以下属性：type：参数的类型（QUERY、BODY、PARAM 等）。metatype：参数的 JavaScript 类型。data：可选的自定义元数据。
  @Injectable()
  transform(value: any, metadata: ArgumentMetadata) {
    console.log('pipe ', value);
    // 在这里进行数据验证和转换
    return value; // 返回经过验证和转换的值
  }
}
```

管道一般用于对参数的验证，用在@Param，或者 @Query 中，第二个参数传入管道类。

```ts
@Controller('cat')
export class CatController {
  @Get()
  findAll(@Query('id', ValidationPipe) id: number) {
    ...
  }
}
```

### 数据传输对象 DTO

DTO 的主要目的是提供一种清晰的方式来定义数据结构，并确保数据的一致性和有效性。

管道可以跟 DTO 结合使用，用来验证 @Body 中的数据结构。

- DTO 可以用于验证传入请求的数据。通过定义输入 DTO，你可以明确地指定哪些字段是必需的，哪些字段是可选的，以及每个字段的验证规则。

- DTO 也可以用于格式化传出响应的数据。通过定义输出 DTO，你可以指定响应的数据结构，并确保响应的数据符合预期的格式。

```sh
pnpm add class-validator class-transformer
```

```ts
import { IsString, IsNumber } from 'class-validator';

export class CreateCatDto {
  @IsString()
  readonly name: string;

  @IsNumber()
  readonly age: number;
}

// 控制器中的使用
import { Controller, Post, Body, UsePipes, ValidationPipe } from '@nestjs/common';
import { CreateCatDto } from './create-cat.dto';

@Controller('cat')
export class CatController {
  @Post()
  @UsePipes(ValidationPipe) // 自动验证
  create(@Body() createCatDto: CreateCatDto) {
    // 在这里使用经过验证的 createCatDto
  }
}
```

## 各种组件的执行顺序

传入请求 -> 中间件 -> 守卫 —> req 拦截器 -> 管道 -> 控制器 -> service -> res 拦截器 -> 异常过滤器 -> 响应

## 附

文章中 demo 的源码：[github](https://github.com/LiShuxue/nest-project-demo)
