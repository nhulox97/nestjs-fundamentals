# Intro to NestJS

`NestJS` es un framework progresivo orientado a construir aplicaciones eficientes y escalables en en el entorno de `Node`. `NestJS` usa arquitectura _out-of-the-box_ lo que significa que el framework nos permite integrar diferentes estrategias y patrones de diseño sin romper la estabilidad e integridad de nuestras aplicaciones.

NestJS surge de la necesidad de estandarizar el desarrollo de aplicaciones del lado del servidor con `Node`, además (pero no limitado) mejorando la experiencia de desarrollo con una integración con `Typescript`, para aquellas personas con experiencia previa en `Express` verán que son los mismos conceptos pero con una visión clara de qué y cómo se están construyendo las cosas.

En este documento repasaremos los principales conceptos dentro del ecosistema de `NestJS`, no sin antes aclarar los pre-requisitos para el correcto seguimiento de esta guía/documento.

## Prerequisites

- `nvm` (Node Version Manager) == 0.39.x
- `Node` == 18.16.x
- `npm` (Node Package Manager) == 9.5.x
- Una terminal (ej: windows terminal. Si usa Linux o macOs usar la terminal nativa).

## Instalación de `nest-cli`

Una de los puntos mas fuerte de `nest` es su `cli` (Command Line Interface) la cual es muy robusta ofreciendo una amplia gama de opciones que nos permitirán crear aplicaciones, agregar módulos, generación de clases, CRUDs, entre otros.

Para instalar esta herramienta necesitamos ejecutar el siguiente comando:

```bash
npm install -g @nest/cli
```

para comprobar la instalación debemos ejecutar el comando `nest --version` y debería de mostrar el siguiente comando:

```bash
nest --version
9.4.2
```

## NestJS Application

Para entender mejor los conceptos básicos de una aplicación NestJS lo primero que haremos es crear nuestra primer aplicación con el siguiente comando:

```bash
nest new nest-fundamentals
```

nos preguntará que administrador de paquetes queremos usar debemos seleccionar `npm`:

```bash
nest new nest-fundamentals
⚡  We will scaffold your app in a few seconds..

? Which package manager would you ❤️  to use?
❯ npm
  yarn
  pnpm
```

ahora solo debemos esperar que todos los paquetes se instalen. La aplicación que se creó debería tener la siguiente estructura:

```
├── README.md
├── nest-cli.json
├── package-lock.json
├── package.json
├── src
│   ├── app.controller.spec.ts
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   └── main.ts
├── test
│   ├── app.e2e-spec.ts
│   └── jest-e2e.json
├── tsconfig.build.json
└── tsconfig.json
```

para motivos de este documento solo nos centraremos en los archivos dentro del directorio `src/`, además debemos de tener en cuenta que el tipo de aplicación que `nest` crea por defecto es una REST API, sin embargo el concepto de controlador puede ser aplicable a otro tipo de arquitecturas donde lo que probablemente cambie sea el nombre de la clase (Listener, Resolver, etc), pero desde un punto de vista de diseño de alto nivel cumplen la misma función.

Una lectura recomendada antes de profundizar en detalles sobre `nest` es entender que es el patrón de diseño `Decorator`, pueden leerlo de acá: https://refactoring.guru/es/design-patterns/decorator.

### Controllers

Los controladores tienen la responsabilidad de ser el punto de entrada en nuestra aplicación para todas las solicitudes que esta pueda recibir por parte de los clientes.

Dentro del directorio `src/` existe el fichero `app.controller.ts` que alberga el siguiente código:

```typescript
import { Controller, Get } from "@nest/common";
import { AppService } from "./app.service";

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

del cual podemos destacar algunas de las cosas más interesantes:

1. El uso de decoradores (`Get`, `Controller`) para modificar el comportamiento de las clases y los métodos de estas.
   - Los decoradores que usa `nest` sirven para definir anotaciones en la metadata de nuestro código, entonces el framework valiéndose del uso de `data reflectors` es capaz de identificar el tipo de clase (en este caso un `Controller`) y cómo esta debe ser configurada a nivel de módulo (`nest module`).
   - En el caso de `Get` como se mencionó antes, esta es una REST API, por lo que se hace uso de los `HTTP Verbs` para el enrutamiento de las solicitudes; también podemos usar otros verbos como: `Post`, `Patch`, `Delete`, `Put`.
2. El servicio se recibe como parte de los parámetros del constructor de la clase del controlador.
   - Los servicios conocidos como `Providers` en el ecosistema de `nest` se valen de una técnica de desarrollo (patrón de diseño) llamada **Dependency Injection** (inyección de dependencias), bajo esta técnica el framework se asegura de crear 1 única instancia de esa clase e inyectarla donde sea que fuese requerida. Se profundizará mas en este tema en la siguiente sección.

### Providers

Este concepto es uno de los mas importante (y más usados) dentro del ecosistema de `nest`, podemos referirnos como `Provider` a cualquier clase que agrupe una serie de funcionalidades y que deba ser consumida (inyectada) ya sea en un controlador o incluso dentro de otro provider.

Si revisamos el proyecto que creamos, dentro del directorio `src/` existe el fichero `app.service.ts` el cual contiene el siguiente código:

```typescript
import { Injectable } from "@nest/common";

@Injectable()
export class AppService {
  getHello(): string {
    return "Hello World!";
  }
}
```

al igual que con nuestra clase `Controller` podemos ver que acá se usa un decorador para definir como `Provider` a nuestra clase, dicho decorador es `Injectable`, si deseamos tratar una clase como un `Provider` es obligatorio decorar nuestra clase con `Injectable`, caso contrario `nest` no reconocerá la clase como tal (aunque la agreguemos a lista de providers) y la aplicación no será ejecutada.

### Modules

Aunque a nivel de lógica de negocio probablemente los módulos sea el lugar donde menos podemos apreciarla, sin embargo es el lugar donde las diferentes piezas que usamos para componer dicha lógica se juntan.

Los `nest Module` son declarativos en el sentido de que la forma en que los configuramos, ya que cada componente de nuestra solución solo puede existir dentro de su contexto (en función del decorador de la clase), con lo anterior el framework se asegura de mantener la integridad de su propia arquitectura.

Si revisamos el proyecto que creamos, dentro del directorio `src/` existe el fichero `app.module.ts` el cual contiene el siguiente código:

```typescript
import { Module } from "@nest/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

el decorador `Module` acepta un objeto como parámetro el cual recibe 3 arrays, aunque lo que esperan estos arrays son clases, cada uno tiene su propia finalidad.

#### `imports`

Ya se mencionó que para que una clase sea considerada como un provider solo debemos agregar el decorador `Injectable` y con eso `nest` se encargará de inyectar dicha dependencia donde sea requerido, sin embargo, el framework espera que sigamos unas reglas de su mismo diseño, y es que un `Provider` solo debería ser declarado como tal (añadido al array de `providers` del module) dentro de su mismo módulo (ModuleA), y en dado caso de ser requerido en otro módulo (ModuleB) lo que deberíamos hacer es definirlo dentro del array `imports` (`ModuleA -> ModuleB.imports`).

De no seguir ese patrón de diseño podemos caer en un problema de dependencias circulares.

#### `controllers`

Son clases relacionadas a los puntos de de entrada de nuestra aplicación, y no se limita a aquellas clases decoradas con `Controller` (ya que eso es algo que aplica a REST API) porque el concepto es algo agnóstico y que puede cambiar (o no) dependiendo del tipo de arquitectura.

#### `providers`

Son clases decoradas con `Injectable`, acá debemos de tener en cuenta los posibles problemas con dependencias circulares (a nivel de `Module || Provider`).

### Running our NestJS application

Si revisamos el archivo `package.json` que esta en el root de nuestra aplicación:

```json
{
  "name": "nest-fundamentals",
  "version": "0.0.1",
  "description": "",
  "author": "",
  "private": true,
  "license": "UNLICENSED",
  "scripts": {
    "build": "nest build",
    "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "node dist/main",
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
    "test:e2e": "jest --config ./test/jest-e2e.json"
  },
  "dependencies": {
    "@nest/common": "^9.0.0",
    "@nest/core": "^9.0.0",
    "@nest/platform-express": "^9.0.0",
    "reflect-metadata": "^0.1.13",
    "rxjs": "^7.2.0"
  },
  "devDependencies": {
    "@nest/cli": "^9.0.0",
    "@nest/schematics": "^9.0.0",
    "@nest/testing": "^9.0.0",
    "@types/express": "^4.17.13",
    "@types/jest": "29.5.0",
    "@types/node": "18.15.11",
    "@types/supertest": "^2.0.11",
    "@typescript-eslint/eslint-plugin": "^5.0.0",
    "@typescript-eslint/parser": "^5.0.0",
    "eslint": "^8.0.1",
    "eslint-config-prettier": "^8.3.0",
    "eslint-plugin-prettier": "^4.0.0",
    "jest": "29.5.0",
    "prettier": "^2.3.2",
    "source-map-support": "^0.5.20",
    "supertest": "^6.1.3",
    "ts-jest": "29.0.5",
    "ts-loader": "^9.2.3",
    "ts-node": "^10.0.0",
    "tsconfig-paths": "4.2.0",
    "typescript": "^4.7.4"
  },
  "jest": {
    "moduleFileExtensions": ["js", "json", "ts"],
    "rootDir": "src",
    "testRegex": ".*\\.spec\\.ts$",
    "transform": {
      "^.+\\.(t|j)s$": "ts-jest"
    },
    "collectCoverageFrom": ["**/*.(t|j)s"],
    "coverageDirectory": "../coverage",
    "testEnvironment": "node"
  }
}
```

en la propiedad de scripts podemos ver una lista de diferentes scripts que nos serán de ayuda al momento de implementar nuevas features, por el momento nos centramos en los scripts:

```json
"start": "nest start",
"start:dev": "nest start --watch",
```

la única diferencia es que `start` únicamente ejecuta la aplicación justo en el estado actual de la misma, mientras que `start:dev` observa por posibles cambios que se puedan agregar. Ahora bien para ejecutar la aplicación ejecutamos el siguiente comando (haciendo uso de npm):

```bash
npm run start
```

deberíamos de ver el siguiente resultado:

```
[Nest] 31872  - 09/14/2023, 10:27:36 PM     LOG [NestFactory] Starting Nest application...
[Nest] 31872  - 09/14/2023, 10:27:36 PM     LOG [InstanceLoader] AppModule dependencies initialized +6ms
[Nest] 31872  - 09/14/2023, 10:27:36 PM     LOG [RoutesResolver] AppController {/}: +11ms
[Nest] 31872  - 09/14/2023, 10:27:36 PM     LOG [RouterExplorer] Mapped {/, GET} route +1ms
[Nest] 31872  - 09/14/2023, 10:27:36 PM     LOG [NestApplication] Nest application successfully started +1ms
```

para comprobar que la aplicación si se esta ejecutando vamos hacer un hit al endpoint que está en `AppController` donde vamos a ver el `string` que retorna el método `getHello` del servicio `AppService`, usaremos `curl` para hacer un Get al `localhost` de nuestro equipo en el puerto `3000` (puerto por defecto para aplicaciones `nest`)

```bash
curl --request GET http://localhost:3000/
```

resultado:

```bash
curl --request GET http://localhost:3000/
Hello World!%
```

## Next steps

En la siguiente sección estaremos hablando sobre el concepto de `Middlewares` y como `nest` los clasifica dependiendo de su funcionalidad y el momento en el que interactúan con la `request` dentro del ciclo de vida de esta.
