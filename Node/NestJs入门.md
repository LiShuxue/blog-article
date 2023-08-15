## NestJS

NestJS (https://docs.nestjs.cn/10/introduction) 是一个基于 TypeScript 的渐进式 Node.js 框架，它结合了 OOP（面向对象编程）、 AOP（面向切面编程）、 DI（依赖注入）和 IoC（控制反转）的最佳实践。

NestJS 提供了一系列强大的组件，使您能够构建可维护、可扩展的应用程序。本文将介绍 NestJS 的基本组件，包括控制器、服务、提供者、模块、中间件、过滤器、管道、拦截器和守卫。

## 控制器（Controllers）

控制器是处理应用程序 HTTP 请求的组件，它们是应用程序的入口点。每个控制器负责处理特定路由下的请求，并将请求逻辑委托给相应的服务。

使用 @Controller('cats') 注解声明一个 controller，处理 cats 路径下的请求。使用 @Get('list') @Put @Post @Delete 来定义请求类型和路径。使用@Param @Body @Query @Headers @Ip() 来获取对应 request 中的信息。

@Query() 用于从查询字符串?xxx=yyy 中提取参数值，而 @Param() 用于从路由参数/user/:id 中提取参数值。

```ts
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  // 当控制器被实例化的时候，CserService会被注入，自动实例化cserService（默认是单例的，如果已经存在，直接返回）
  constructor(private readonly catService: CatService) {}

  @Get()
  findAll() {
    const catList = this.catService.findAll();
    return catList;
  }
}
```

## 提供者（Providers）&& 服务（Services）

通过将 @Injectable() 装饰器应用于类，您可以将其标记为一个提供者。提供者会被声明在@Module 的 providers 中，可被依赖注入。

服务是处理应用程序业务逻辑的组件。它们处理从控制器传递的数据，并执行特定的操作。通过依赖注入的方式在控制器中进行实例化和使用。

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class CatsService {
  findAll() {
    // 这里可以调用数据库方法去查询数据库并返回
    return [];
  }
}
```

## 模块（Modules）

模块是代码组织的基本单元。每个应用程序都由一个或多个模块组成，至少有一个根模块。模块在 NestJS 中负责将应用程序的不同部分组合在一起。

通过 @Module 装饰器将元数据附加到模块类中。@Global 装饰器可以使模块成为全局作用域。

- imports 字段定义了当前模块所依赖的其他模块。导入其他模块的功能意味着您可以访问这些模块中的控制器、服务、提供者等。

- controllers 字段用于指定当前模块中包含的控制器。

- providers 字段定义了当前模块中提供的服务、提供者和其他可注入的对象。提供者可以是服务、工厂、库等。

- exports 字段用于将当前模块的功能导出到其他模块中。通过将模块的功能导出，其他模块可以使用当前模块中定义的控制器、服务等。

```ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  imports: [],
  controllers: [CatsController],
  providers: [CatsService],
  exports: [],
})
export class CatsModule {}
```

## 中间件（Middlewares）

中间件是在路由处理程序 <b>之前</b> 调用的函数。中间件函数可以访问请求和响应对象，以及应用程序请求响应周期中的 next() 中间件函数。中间件可以用来处理请求、进行验证、日志记录、实现认证等操作。

中间件可以使用类的形式，也可以是函数形式。

```ts
// 1. 使用类的形式声明中间件
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...', req);
    // 执行下一个中间件或路由处理程序
    next();
  }
}

// 2. 函数形式
export function logger(req, res, next) {
  console.log(`Request...`);
  next();
}
```

中间件也是用 @Injectable() 声明，但是中间件不能在 @Module() 装饰器中列出。我们必须使用模块类的 configure() 方法来设置它们。包含中间件的模块必须实现 NestModule 接口。

apply 方法来使用中间件，可以使用多个。可以使用在特定的路径 forRoutes('cats')、特定的控制器 forRoutes(CatsController)，或者是所有路径 forRoutes('\*')中。exclude() 方法可以排除某些路由。

```ts
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware) // 可以使用多个中间件
      .forRoutes('cats'); // 可以使用在特定的路径forRoutes('cats')，或者特定的控制器forRoutes(CatsController)，或者是所有路径forRoutes('*')
  }
}
```

## 守卫（Guards）

守卫也是一个使用 @Injectable() 装饰器的类。守卫应该实现 CanActivate 接口，必须实现一个 canActivate() 函数。此函数应该返回一个布尔值，用于指示是否允许当前请求。

守卫在每个中间件之后执行，但在任何拦截器或管道之前执行。一般用来做鉴权，来确定请求是否可以继续。

```js
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}
```

守卫可以是控制使用范围的，方法范围 @UseGuards() 或全局范围 app.useGlobalGuards 使用。

```js
// 方法范围
@Controller('cats')
@UseGuards(AuthGuard)
export class CatsController {}

// 全局范围
const app = await NestFactory.create(AppModule);
app.useGlobalGuards(new AuthGuard());
```

## 拦截器（Interceptors）

拦截器也是使用 @Injectable() 装饰器的类。拦截器应该实现 NestInterceptor 接口，并且实现 intercept 方法。

拦截器用于 在请求到达控制器之前或响应返回客户端之前 添加额外的逻辑。它们可以用于修改请求或响应对象、记录日志等。

next.handle() 返回一个 Observable，此流包含从路由处理程序返回的值（响应）, 我们可以使用 map() 运算符地对其进行改变。

```ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before request...');
    console.log(context);
    return next.handle().pipe(
      map((data) => {
        console.log('After response...');
        console.log(data);
        // 在这里对响应数据进行转换或操作
        return data; // 返回转换后的数据
      })
    );
  }
}
```

@UseInterceptors 可以将拦截器使用在局部方法上，也可以 通过 app.useGlobalInterceptors 使用在全局。可以传多个拦截器，按顺序处理。

```js
// 局部
@Controller('cats')
@UseInterceptors(LoggingInterceptor)
export class CatsController {}

// 全局
const app = await NestFactory.create(ApplicationModule);
app.useGlobalInterceptors(new LoggingInterceptor());
```

## 管道（Pipes）&& 数据传输对象 DTO

管道也是具有 @Injectable() 装饰器的类。管道应实现 PipeTransform 接口，且实现 transform 方法。

可以应用于控制器的路由处理程序中，用于在数据传递给控制器之前对其进行验证、转换和处理。管道可以确保传递给控制器的数据的完整性、一致性和有效性。

验证管道要么返回该值，要么抛出一个错误。

```js
import { Injectable, PipeTransform, ArgumentMetadata } from '@nestjs/common';

@Injectable()
export class ValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    console.log(value);
    // 在这里进行数据验证和转换
    return value; // 返回经过验证和转换的值
  }
}
```

管道一般用于对参数的验证，用在@Param，或者 @Query 中，第二个参数传入管道类。

```js
export class UserController {
  @Get(':id')
  getUser(@Param('id', ValidationPipe) id: number) {
    // todo
  }
}
```

管道也可以跟 DTO 结合使用，用来验证@Body 中的数据结构。

```js
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

@Controller('cats')
export class CatsController {
  @Post()
  @UsePipes(ValidationPipe)
  create(@Body() createCatDto: CreateCatDto) {
    // 在这里使用经过验证的 createCatDto
  }
}
```

## 过滤器（Filters）

过滤器（Filters）是用于处理异常的组件，它们允许您在应用程序中统一处理抛出的异常，以提供更友好和一致的错误响应。通常是使用 @Catch() 装饰器进行注解的类。

能够在全局范围或特定控制器/路由范围内捕获异常，并根据异常类型提供自定义的响应。

它是响应返回给客户端的最后一次过滤。

```js
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

过滤器可以通过 UseFilters 使用在某个路径上，也可以 app.useGlobalFilters 用在全局。可以传入多个过滤器。

```js
//局部
@Controller('cats')
@UseFilters(HttpExceptionFilter)
export class CatsController {}

// 全局
const app = await NestFactory.create(AppModule);
app.useGlobalFilters(new HttpExceptionFilter());
```

## 各种组件的执行顺序

请求 -> 中间件 -> 守卫 —> req 拦截器 -> 管道 -> 控制器方法执行 -> service 方法执行 -> 控制器返回响应 -> res 拦截器 -> 异常过滤器 -> 响应
