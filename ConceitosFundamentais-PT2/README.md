# Conceitos Fundamentais - Anotações

Nesse README trago anotações sobre o meu entendimento sobre alguns temas abordados no curso, e que são fundamentais para o entendimento do framework.

## Pipes

Traduzindo para português "Tubos", os Pipes normalmente são usados para transformar os dados que passam por eles ou para validá-los.

No Nest os Pipes são classes como o decorator `@Injectable()` e assim irá obrigar a declaração de um método chamado `transform()` que irá receber o valor que está sendo passado e irá retornar o valor transformado.

Exemplo de uso:

```typescript
@Get(':id')
async show(@Param('id', ParseIntPipe) id: number) {
  return this.userService.show(id);
}
```

Nesse exemplo, o `ParseIntPipe` irá transformar o valor passado para um inteiro.

## Interceptors

É uma técnica inspirada na AOP (Aspect Oriented Programming) que permite adicionar lógicas extras antes ou depois da execução de um método. Transformando o resultado retornado de uma função, ou até mesmo interrompendo a execução de um método.

No Nest é feito pela interface `NestInterceptor` e que irá obrigar a declaração do método `intercept()`, que irá receber o contexto da requisição `(ExecutionContext)` e o manipulador da chamada final `(CallHandler)`.

### Como usar os Interceptors?

Os interceptors podem ser usados em um controle, método ou globalmente. O decorator dele é o `@UseInterceptors()`, mas também pode usar o método `useGlobalInterceptors()` no `main.ts` para usar globalmente.

Exemplo de uso:

user.controller.ts

```typescript
import {
  // UseInterceptors,
} from '@nestjs/common';
import { CreateUserDTO } from './dto/create-user.dto';
import { UserService } from './user.service';
// import { LogInterceptor } from 'src/interceptors/log.interceptor';

// @UseInterceptors(LogInterceptor) -> Pode ser aplicado no controler
@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  // @UseInterceptors(LogInterceptor) -> Pode ser aplicado aqui diretamente
  @Post()
  async create(@Body() data: CreateUserDTO) {
    return this.userService.create(data);
  }

 ...
```

main.ts:

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ValidationPipe } from '@nestjs/common';
import { LogInterceptor } from './interceptors/log.interceptor';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
  app.enableShutdownHooks();
  app.useGlobalInterceptors(new LogInterceptor()); //Aqui aplica o interceptor globalmente
  await app.listen(3000);
}
bootstrap();
```

Exemplo de um código Interceptor:

```typescript
import { CallHandler, ExecutionContext, NestInterceptor } from '@nestjs/common';
import { Observable, tap } from 'rxjs';

export class LogInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const dt = Date.now();
    return next.handle().pipe(
      tap(() => {
        const request = context.switchToHttp().getRequest();

        console.log(`URL: ${request.url}`);
        console.log(`Execução levou: ${Date.now() - dt}ms`);
      }),
    );
  }
}
```

Nesse exemplo, o `LoggingInterceptor` irá ser executado antes do método `findAll()`.

## Middleware

Os middlewares são funções que são executadas antes do manipulador de rotas, e podem ser usadas para manipular a requisição antes de chegar no controlador, ou até mesmo interromper a requisição.

Essas funções podem ser declaradas em uma classe com o decorador `@Injectable()` que irá implementar a interface `NestMiddleware` para obrigar a declaração do método `use()`.

Essas funções acessam objetos `Request` e `Response` e a função `next()` que chama a próxima função middleware ou o manipulador de rotas. Dessa maneira seguimos o design pattern **`Chain of Responsibility (Cadeia de responsabilidade)`**.

### Por que usar Middleware?

Com o uso dos middlewares iremos executar qualquer código, alteando os objetos de requisição e resposta, ou até mesmo interrompendo a requisição já que as funções middleware são chamados sempre antes de uma rota ser executada.

Exemplo de uso:

- Middleware

```typescript
import { BadRequestException, NestMiddleware } from '@nestjs/common';
import { NextFunction, Request, Response } from 'express';

export class UserIdCheckMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('UserIdCheckMiddleware', 'antes');
    if (isNaN(Number(req.params.id)) || Number(req.params.id) < 0) {
      throw new BadRequestException('ID Inválido');
    }
    console.log('UserIdCheckMiddleware', 'depois');
    next();
  }
}
```

- Chamando um Middleware no modulo

```typescript
import {
  MiddlewareConsumer,
  Module,
  NestModule,
  RequestMethod,
} from '@nestjs/common';
import { UserController } from './user.controller';
import { UserService } from './user.service';
import { PrismaModule } from 'src/prisma/prisma.module';
import { UserIdCheckMiddleware } from 'src/middlewares/user-id-check.middleware';

@Module({
  imports: [PrismaModule],
  controllers: [UserController],
  providers: [UserService],
  exports: [],
})
export class UserModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(UserIdCheckMiddleware).forRoutes({
      path: 'users/:id',
      method: RequestMethod.ALL,
    });
  }
}
```

## Guards

São classes com o decorator `@Injectable()` que implementam a interface `CanActivate` e que obrigam a declaração do método `canActivate()`. Eles tem a responsabilidade de determinar se uma solicitação Request deve ser manipulada por um manipulador de rota.

Com os Guards podemos proteger rotas, verificar se o usuário está autenticado, se tem permissão para acessar a rota, ou até mesmo verificar se o usuário tem permissão para acessar um recurso.

## Exceptions

São erros que ocorrem durante a execução de um programa, e que podem ser tratados para que o programa não pare de funcionar. Elas são capturadas automaticamente por uma camada de aplicação que transforma as exceções em mensanges amigáveis para o usuário.

Essas mensagens tem o padrão de serem retornadas em JSON contendo o status da requisição e a mensagem de erro.

Apesar de podermos usar as exceções já existentes do Nest, podemos criar exceções personalizadas para o nosso projeto, para isso deve ser criado uma classe que estende a classe `HttpException`

### Exceptions Filters

Podemos criar filtros de exceções que são classes com o decorator `@Catch(T)` que implementam a interface `ExceptionFilter` e que obrigam a declaração do método `catch()` que nos dá acesso a exceção e o host que pode nos fornecer o contexto da execução com o `Request` e `Response`.

Exemplo de uso:

```typescript
  async exists(id: number) {
    if (
      !(await this.prisma.user.count({
        where: {
          id,
        },
      }))
    ) {
      throw new NotFoundException(`O usuário ${id} não existe`); // Aqui é lançado a exceção
    }
  }
```

## Param Decorators

O Nest oferece um conjunto de decoradores de parâmetros úteis que podem ser usados para acessar os parâmetros da solicitação. Eles são usados para acessar os parâmetros da solicitação, como o corpo da solicitação, os cabeçalhos da solicitação, os parâmetros da solicitação, etc.

### Custom Decorators

Nós podemos criar os nossos decoradores, isso nos possiblita por exemplo a criação de um decorator que irá pegar o usuário autenticado e colocar no objeto de requisição, permissões que o usuário tem, quando um usuário está autenticado, etc.

Exemplos de uso:

Um exemplo com o do Nest:

```typescript
  @Get(':id')
  async show(@Param('id', ParseIntPipe) id: number) {
    return this.userService.show(id);
  }
```

Um exemplo de um decorator customizado:

```typescript
import { ExecutionContext, createParamDecorator } from '@nestjs/common';

export const ParamId = createParamDecorator(
  (_data: unknown, context: ExecutionContext) => {
    return Number(context.switchToHttp().getRequest().params.id);
  },
);
```

E o uso dele:

```typescript
  @Get(':id')
  async show(@ParamId() id: number) {
    return this.userService.show(id);
  }
```
