#Create an API Gateway and microservice using NestJS TypeScript.
<p>A simple gateway connecting services. Nothing Fancy just the necessities.</p>


##Make project directory
```shell
mkdir nest-test-api-gateway

cd nest-test-api-gatewy
```
##Install nestjs/cli tools
```shell
yarn global add @nestjs/cli
```

##Create new nest project called service-a
```shell
nest new service-a

cd service-a/
```
##Remove unneeded test files (optional)
```shell
rm ./src/app.controller.spec.ts ./src/app.service.ts
```

##Install nest micro service packages
```shell
yarn add @nestjs/microservices
```

##Update main.ts to use microservice 
>./service-a/src/main.ts
```javascript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { Transport } from '@nestjs/microservices';
import { Logger } from '@nestjs/common';

const logger = new Logger();

async function bootstrap() {
    const app = await NestFactory.createMicroservice(AppModule, {
        transport: Transport.TCP,
        options: {
            host: '127.0.0.1',
            port: 8888,
        },
    });
    app.listen(() => logger.log('Microservice A is listening'));
}
bootstrap();

```
##Update App controller to use microservice 
>./service-a/src/app.controller.ts
```javascript
import { Controller } from "@nestjs/common";
import { MessagePattern } from "@nestjs/microservices";
import { of } from "rxjs";
import { delay } from "rxjs/operators";

@Controller()
export class AppController {
  @MessagePattern({ cmd: "ping" })
  ping(_: any) {
    return of("pong").pipe(delay(1000));
  }
}
```
##create api gateway for services
```shell
cd ../

nest new api-gateway && cd api-gateway
```
##Remove unneeded test file(optional)
```shell
rm ./src/app.controller.spec.ts
```
##Install nest micro service packages
```shell
yarn add @nestjs/microservices
```

##Register the microservice with App module 
>./api-gateway/src/app.module.ts
```javascript
@Module({
  imports: [
    ClientsModule.register([
      {
        name: "SERVICE_A",
        transport: Transport.TCP,
        options: {
          host: "127.0.0.1",
          port: 8888
        }
      }
    ])
  ],
  controllers: [AppController],
  providers: [AppService]
})
```
##Update api gateway app service to use microservice 
>./api-gateway/src/app.service.ts
```javascript
import { Injectable, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { map } from 'rxjs/operators';

@Injectable()
export class AppService {
  constructor(
    @Inject('SERVICE_A') private readonly clientServiceA: ClientProxy,
  ) {}

  pingServiceA() {
    const startTs = Date.now();
    const pattern = { cmd: 'ping' };
    const payload = {};
    return this.clientServiceA
      .send<string>(pattern, payload)
      .pipe(
        map((message: string) => ({ message, duration: Date.now() - startTs })),
      );
  }
}
```
##Update the @Get decorator in api-gateway app controller to handle the new service
>./api-gateway/src/app.controller.ts
```javascript
@Get('/ping-a')
  pingServiceA() {
    return this.appService.pingServiceA();
  }
```

