### Introduction 

NestJS(이하 Nest)는 효율적인, 확장성을 갖춘 Node.js 서버사이드 어플리케이션을 빌딩하기 위한 프레임워크입니다. Nest는 진보적인 JavaScript(이하 JS)를 사용하긴 하나, Typescript(이하 TS)로 빌드 되었으며, TS를 완전하게 지원합니다(물론, 여전히 퓨어 JS를 사용한 개발도 가능합니다.). 그리고 Nest는 OOP(Object Oriented Programming), FP(Functional Programming), 그리고 FRP(Funtional Reactive Progrmaming)의 요소들의 조합이기도 합니다. 

내부적으로 Nest는 Express(기본값)와 같은 강력한 HTTP 프레임워크를 사용해 만들어졌으며, 선택적으로 Fastfy 또한 사용해 설정하는 것이 가능합니다. 

Nest는 일반적인 Node.js 프레임워크들 위에서 더 높은 추상화단계를 제공해주며, 동시에 개발자들에게 직접적으로 API들을 노출시켜 줍니다. 이 점은 개발자들에게 플랫폼 아래에서 이용 가능한 수많은 서드파티 모듈에서의 사용이 가능하다는 자유를 제공합니다.

#### Philosophy

최근 들어, Node.js의 덕분으로 JS는 프론트, 백엔드 어플리케이션 양쪽을 위한 웹의 '랑그아 프란카'와 같은 존재로 만들어 주었습니다. 이점은 Angular, React, Vue 와 같이 빠르고, 테스트 가능하며, 확장성이 높은 프론트엔드 어플리케이션 프로젝트를 만들어냈습니다. 하지만 수많은 라이브러리, helper들, 툴들은 Node를 위해 존재하긴 하나, 아주 주요한 메인 문제, 아키텍쳐와 관련한 효과적인 문제 해결책은 나오지 않은 상태입니다.

Nest는 이러한 상황에서 최신의 어플리케이션 아키텍쳐를 제공하고, 이는 개발자들, 그리고 팀에서 테스트하기 용이하고, 수평적 확장이 쉽고, 느슨한 결합을 가지고 있으며, 마지막으로 관리가 용이합니다. 이 아키텍쳐는 기본적으로 Angular에서 참고해서 구현되어 있습니다.

#### Installation
시작하기 위해, 당신이 프로젝트를 시작하기 위한 발판을 Nest CLI가 준비해봅시다. 또는 스타터 프로젝트를 간단히 클론 받아도 됩니다. (양쪽 모두 동일한 결과물을 제공합니다. )

발판을 만들기 위해 NestCLI를 통해 명령어를 입력해봅시다. 이 명령어를 통해 새로운 프로젝트의 디렉터리와 당신의 프로젝트를 위한 컨벤션적인 기본 구조를 생성, 지원 모듈 그리고 최초 코어 Nest 파일들을 채워 넣습니다. NestCLI와 함께하는 신규 프로젝트를 생성은 최초 유저들에게 추천 드립니다. 우리는 `First Steps` 안에서 이러한 접근법을 시도해볼 것입니다. 

```shell
$ npm i -g @nestjs/cli
$ nest new project-name
```

> info Hint TS 프로젝트와 stricter feature set을 함께 쓰길 원한다면 `--strict` 플래그 명령어에 넣고 `nest new` 명령어를 사용하면 된다. 

#### Alternative
위에서 설치하는 방식의 대안으로 Git 을 통해 TypeScript starter 프로젝트를 설치하는 방법도 있습니다. 

```shell
$ git clone https://github.com/nestjs/typescript-starter.git project
$ cd project
$ npm install
$ npm run start
```

> 만약 git 히스토리 없이 레포지터리를 클론 받고 싶다면, degit 을 사용하십시오.

이제 브라우저를 열고 `http://localhost:3000/` 을 찾아 가봅시다. 

만약 JS 버전의 스타터 프로젝트를 설치하길 원한다면, `javascript-start.git`을 명령어 순서 중에 사용하시면 됩니다. 

또한, npm 혹은 yarn과 같은 패키지 매니저를 통해, 코어와 지원하는 파일들을 직접 골라, 새로운 프로젝트 생성을 수동으로 해볼 수도 있습니다. 이 경우, 프로젝트 첫 문서를 비롯해서 모두 본인이 작성해야 합니다. 

```shell
$ npm i --save @nestjs/core @nestjs/common rxjs reflect-metadata
```