### First steps 
이번 장에서는, Nest의 **core fundamental** 들을 배울 것입니다. Nest 어플리케이션의 핵심 블록들에 친숙해지기 위해, 우리는 기초적인 수준으로 저변에 깔린 기능 기본적인 CRUD 어플리케이션을 빌드해볼 것입니다. 

#### Language
우리는 TypeScript(이하 TS)를 사랑하고 있습니다. 그러나 무엇보다도 사실 우린 Node.js(이하 Node)를 사랑하지요. 그렇기에 Nest는 TS와 pure JavaScript를 동시에 호환성을 갖고 있는 이유일 겁니다. Nest는 최신 언어의 기능들에서 이점을 취하고 있습니다. 이를 위해 Nest를 바닐라 JS와 사용할 수 있기 위해, Babel 컴파일러를 사용합니다. 

우리는 대게 TS를 예시 코드로 제공할 것입니다. 하지만 스니펫을 누르시면 항상 바닐라 JS 코드도 볼 수 있습니다.(당연히, 해당 글에서는 제공하지 않습니다. TS 짱...ㅎㅎ)

#### 선행 조건 
Node.js(>= 16)를 OS에 설치해두십시오.

#### set up
새로운 프로젝트를 구성하기 위해선 Nest CLI가 필요합니다. npm을 통해 인스톨할 수 있으며, 인스톨 후 새로운 Nest 프로젝트를 당신의 OS 터미널 상에 입력함으로 구축할 수가 있습니다. 

```shell
$ npm i -g @nestjs/cli
$ nest new project-name
```

> HINT 
> 새로운 프로젝트를 TS의 stricter 기능과 함께 시작하고 싶으시면 `--strict` 플래그를 `nest new` 커맨드 에 붙이시면 됩니다. 

`project-name` 이라는 디렉토리가 생성되면서, 노드 모듈들, 그리고 몇가지 다른 자동으로 설치되는 파일들이 보일 것입니다. 그리고 `src/` 디렉토리는 몇 개의 핵심 파일들을 생성된 뒤 채워져 있을겁니다. 

```plain
src
├─ app.controller.spec.ts
├─ app.controller.ts
├─ app.module.ts
├─ app.service.ts
└─ main.ts
```

구성요소들을 설명하면 다음과 같습니다. 
- `app.controller.ts` : 단일 루트에 대한 기본적인 컨트롤러
- `app.controller.spec.ts` : 기본 컨트롤러에 대한 유닛 테스트
- `app.module.ts` : 어플리케이션의 루트 모듈
- `app.service.ts` : 단일 메서드를 가진 기본 서비스
- `main.ts` : Nest 어플리케이션의 인스턴스를 생성하기 위한 핵심 기능인 `NestFactory`를 사용하기 위한 어플리케이션의 엔트리 파일 

`main.ts` 는 우리의 어플리케이션을 **bootstrap** 해줄 비동기 기능이 포함되어 있습니다. 

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```
Nest 어플리케이션 인스턴스를 생서하기 위해서, 우리는 `NestFactory`라는 핵심 클래스를 사요한다. `NestFactory`는 어플리케이션 인스턴스를 생성하는 것을 허락해주는 몇개의 정적 메서드를 포함하고, 접근 가능합니다. `create()` 메서드는 어플리케이션 객체를 반환하고, 이는 `INestApplication` 인터페이스로 채워져 있습니다. 이 객체는 앞으로 배울 챕터들에서 설명할 다양한 메서드들을 제공합니다. 위의 `main.ts`예시 코드를 보더라도, 우리는 간단하게 HTTP 수신자 기능을 부트업 하는 것으로 보여주며, 이는 어플리케이션이 HTTP 요청의 인바운드를 기다리도록 만들어줍니다. 

주목할 지점은 Nest CLI를 통해 스캐폴드된 프로젝트가 초기의 프로젝트의 구조를 형성하고, 이는 개발자들이 각기 다른 디렉토리 상의 각 모듈들에 대해서도 컨벤션을 따르도록 유도한다는 점입니다. 

> HINT
> 기본적으로, 당신의 어플리케이션을 생성하는 동안 발생하는 에러는 `exit`코드 1을 받습니다. 만약 error에 대해 옵션을 끄는 대신에 throw 하고 싶다면 `abortOnError` (e.g. `NestFactory.create(AppModule, { abortOnError: false })`).

#### Platform 
Nest는 플랫폼에 구애받지 않는 프레임워크를 지향합니다. 플랫폼 독립성은 다양한 종류의 어플리케이션들 사이에서 이점을 개발자들이 취할 수 있는 논리적 부분에 대해 재사용성을 가능케 만들어줍니다. 기술적으로, Nest 는 어뎁터만 정상적으로 생성된다면, 어떤 Node 기반 HTTP 프레임워크들을 가지고 구동이 가능합니다. 단, 기본적으로 공식적으로 지원하는 것은 `express` 와 `fastify`를 지원합니다. 당신의 필요에 따라 가장 적절한 쪽을 고르시면 됩니다. 

- `platform-express` : `Express`는 익히 알려진 미니멀리스틱한 노드 기반 웹 프레임워크입니다. 기본적으로 매우 디테일하게 검증되었으며, 프로적션-레디 상태의 라이브러리로 커뮤니티에 의해 구현된 리소스들이 상당합니다.`@nestjs/platform-express`패키지는 기본값으로 사용됩니다. 다수의 Express에 종사한 사용자들에게는 특별히 더 할것이 없을 정도입니다. 
- `platform-fastify` : `Fastify` 는 높은 성능과 낮은 오버헤드를 특징으로 하는 프레임워크이며, 최대 효율성과 속도를 중시하는 플랫폼입니다. 

`NestFactory.create()` 메소드를 타입하여 당신이 지나치는 순간, 아래의 예시처럼 `app` 객체는 특정 플랫폼을 위한 베타적인 메서드를 가지게 된다. 그러나 실제로 기본 플랫폼 API에 액세스하려는 경우가 아니면 유형을 지정할 필요 가 없습니다 .

```typescript
const app = await NestFactory.create<NestExpressApplication>(AppModule);

```

####  어플리케이션 실행
일단 설치가 마무리 된 뒤, OS의 프롬프트에서 다음 명령어를 실행시킴으로 HTTP 요청을 받기 위해 리스닝하는 어플리케이션을 실행 시킬 수 있습니다.

```shell
$ npm run start
```

> HINT 
> 개발 프로세스 속도 향상을 시키기 위해 다음 플래그를 통해`SWC builder` 를 사용할 수 있다. `npm run start -- -b swc`

```plain
Speedy Web Compiler 는 Rust-based 플랫폼으로 컴파일과 빌드 속도를 월등하게 만들어줍니다. 실제로 적용해본 결과 실행 속도 면에서 차이는 없으나, 컴파일 속도는 월등하게 빨랐습니다. 이 부분은 엄청 도움이 되거나, 프로덕션 환경에서 성능적 이점이 있는 것은 아니지만, 적용 해놓는 것만으로도 개발 시 편리함 + 배포 시 빠른 배포가 가능하다는 점 등에선 매우 유용해 보입니다. 
```

위의 명령어를 통해 `src/main.ts` 파일에 기제된 포트 번호로 HTTP 서버 리스닝이 수행되며 앱이 실행 됩니다. 일단 어플리케이션이 실행 되면, 브라우저에서 `http://localhost:3000/` 으로 접속이 가능하고 `Hello World!` 메시지를 볼 수 있습니다. 

파일의 변화를 감시하면서, 어플리케이션을 지속적으로 재 실행하는 방법은 

```shell
$ npm run start:dev
```

이 명령어는 파일들을 감시하고, 자동적으로 서버의 재 컴파일 및 리로드를 제공할 것입니다. 

#### Linting and formatting
Nest CLI 는 대규모로 안정적인 개발 워크플로를 구축하기 위해 최선의 노력을 다하고 있습니다. 그래서 생성된 Nest 프로젝트는 linter, formatter 둘다 사전 설치를 진행해줍니다. 각각 `eslint`, `prettier`입니다.

> Lint vs prettier
> 프리티어는 포맷팅 룰을 개선해주는 역할로 프로그래머의 실수를 가능한 줄여주는 역할을 하며, 코드의 수준을 보안해주는 역할이 Linter이 해줍니다.

최대한의 안정성과 확장성을 보장하기 위해, Nest는 두 패키지를 제공해주며, 이 설정을 사용하면 설계 상 공식 확장과 깔끔한 IDE 통합 환경을 구축하는데 도움을 줍니다. 

IDE가 관련되지 않은 헤드리스 환경의 경우 Nest 프로젝트에는 즉시 사용 가능한 npm 스크립트도 내장하고 있습니다. 

```shell
# Lint and autofix with eslint
$ npm run lint 

# Format with prettier 
$ npm run format
```