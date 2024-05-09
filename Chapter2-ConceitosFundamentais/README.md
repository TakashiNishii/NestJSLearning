# Conceitos Fundamentais - Anotações

Nesse cápitulo trago anotações sobre o meu entendimento sobre alguns temas abordados no curso, e que são fundamentais para o entendimento do framework. (Esse cápitulo é mais teórico)

## Bootstrap

Apesar do nome, não tem nada a ver com o famoso do frontend, não envolve nada de CSS. O Bootstrap do NestJS é um auxiliador para iniciar a aplicação, ele é um módulo que inicializa a aplicação, e é responsável por configuração das nossas definições de módulos, controladores, serviços, etc.

Exemplo:

```typescript
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

async function bootstrap() {
  // Cria a aplicação com o módulo principal
  const app = await NestFactory.create(AppModule);
  // Porta que a aplicação vai rodar
  await app.listen(3000);
}
bootstrap();
```

## Decorators

Recurso de anotação do TypeScript que modifica o componente, classe ou objeto em tempo de execução. O NestJS tem muitos decorators úteis para definir rotas, parâmetros, tipos de requisição, etc. E nós também podemos criar nossos próprios decorators.

Exemplo:

```typescript
import { Module } from "@nestjs/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

Como podemos ver, o `@Module` é um decorator que modifica a classe `AppModule` em tempo de execução. E também esse decorator é um pronto do NestJS que é utilizado para definir um módulo. e seus argumentos é um objeto que contém as configurações do módulo (imports, controllers, providers) e que serão explicados em outra sessão.

:warning: **Observação**:

- Os decorators são chamados sempre uma linha acima do componente que ele está modificando, e sempre precedidos por um `@`.

- Podemos ter dois ou mais decorators em um mesmo componente, e eles são executados na ordem que são declarados.

## Modules

Trazendo o conceito do AngularJS, os módulos é a divisão de sessões da nossa aplicação. Cada módulo é responsável por uma parte da aplicação, por exemplo, definir módulos para usuário e seus dados, outro para produtos, arquivos e etc, podendo compartilhar entre si os recursos.

:warning: **Observação**: O `import` do código anterior, é onde importamos outro módulo para o módulo que estamos criando, ganhando acesso aos recursos do módulo importado. E o `exports` é onde exportamos os recursos do módulo que estamos criando, para que outros módulos possam acessar.

## Controllers

Para explicar um controller existe uma analogia muito boa, no qual o controller funciona como um garçom 🤵 de um restaurante, ele recebe o pedido do cliente (requisição) 📄, analisa e valida o pedido, e envia para o cozinheiro (serviço) 👨‍🍳 preparar o prato (resposta)🍝.

Parte da aplicação que lida com as requisições HTTP, ele é responsável por receber as requisições, processar e devolver uma resposta. Ele é o intermediário entre o cliente e o servidor.

Exemplo:

```typescript
import { Controller, Get, Post } from "@nestjs/common";
import { AppService } from "./app.service";

//De parâmetro podemos colocar uma string que indica o prefixo da rota
@Controller() // -> 🤵
export class AppController {
  // Criamos um construtor que recebe o serviço
  constructor(private readonly appService: AppService) {}

  @Get() // -> 📄
  getHello(): string {
    // -> 🍝
    return this.appService.getHello(); // -> 👨‍🍳
  }

  @Post() // -> 📄
  setHello(): string {
    // -> 🍝
    return "POST: Hello Takas!"; // -> 👨‍🍳
  }
}
```

## Services 👨‍🍳

O responsavel por toda a lógica de negócio da aplicação, ele é o cozinheiro que prepara o prato, ele é responsável por toda a regra de negócio da aplicação, e é chamado pelo controller para realizar as operações.

Se não seguir os conceitos de SOLID, o service pode se tornar um **"God Object"**, ou seja, um objeto que faz tudo, e isso é ruim, pois ele se torna muito grande e difícil de manter.

Exemplo:

```typescript
import { Injectable } from "@nestjs/common";

//Decorator que indica que a classe é um serviço, podemos colocar no parâmetro o nome do serviço
@Injectable()
export class AppService {
  getHello(): string {
    return "Hello World!";
  }
}
```
