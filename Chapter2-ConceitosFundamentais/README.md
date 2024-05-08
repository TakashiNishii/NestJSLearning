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
