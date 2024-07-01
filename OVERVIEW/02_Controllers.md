### Controllers 
Controller들은 들어오는 request 들을 처리학, response들을 클라이언트에게 반환해주는 책임을 가지고 있습니다. 

![](../src/02_01.png)

Controller 의 목적은 어플리케이션을 위한 세부 request를 수신하는 것이다. 어느 요청이든지 컨트롤러는 수신하고 `routing` 메커니즘이 이를 제어한다. 종종 각 Controller 들은 하나의 루트 이상을 가질 수 있고, 다른 루트들은 다른 action들을 수행할 수 있다. 

> HINT
> 빠른 CRUD 컨트롤러를 내장된 validation 과 같이 생성하고 싶다면 `nest g resource [name]` 으로 생성해보세요.

#### Routing
다음 예시는 `@Controller()` 데코레이터를 사용하며, 이는 Controller라는 것을 정의내리는데 필요합니다. 우리는 `cats`라는 접두사를 가진 추가적인 루트를 만들 수 있다. `@Controller()` 안에 패스 접두사를 사용함으로써, 우리는 쉽게 관련된 루트들을 그룹 지을 수 있으며, 반복적인 코드를 최소화 하는 것이 가능합니다. 예를 들어 `/cats` 루트를 기반으로 cat 엔티티를 상호작용으로 관리하는 루트의 일련의 그룹을 만들 수 있습니다. 이는 파일 안에 각 루트에 대해 반복적인 부분을 최소화 하는 것이 가능합니다. 

```typescript
// cats.controller.ts 
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
```

> HINT 
> CLI를 가지고 컨트롤러를 생성하고 싶다면, 간단하게 `nest g controller [name]` 이라고 명령을 입력을 수행하면 됩니다. 

`findAll()` 메서드 앞의 `@Get()` HTTP 요청 메서드 데코레이터는 Nest에게 HTTP 요청에 대한 특정 엔드포인트에 대한 핸들러를 생성하라고 지시합니다. 엔드포인트는 HTTP 요청 방법(이 경우 GET) 및 경로에 해당합니다. 경로란 핸들러의 경로는 컨트롤러에 대해 선언된 (선택적) 접두사와 메서드의 데코레이터에 지정된 경로를 연결하여 결정됩니다. 모든 경로(cats)에 대한 접두사를 선언했고 데코레이터에 경로 정보를 추가하지 않았으므로 Nest는 GET /cats 요청을 이 핸들러에 매핑합니다. 언급한 대로 경로에는 선택적 컨트롤러 경로 접두사와 요청 메서드 데코레이터에 선언된 경로 문자열이 모두 포함됩니다. 예를 들어 데코레이터 `@Get('breed')`와 결합된 cats의 경로 접두사는 `GET /cats/breed`와 같은 요청에 대한 경로 매핑을 생성합니다.

위의 예에서 이 엔드포인트에 GET 요청이 이루어지면 Nest는 요청을 사용자 정의 findAll() 메서드로 라우팅합니다. 여기서 선택한 메소드 이름은 완전히 임의적입니다. 분명히 경로를 바인딩할 메서드를 선언해야 하지만 Nest는 선택한 메서드 이름에 아무런 의미를 부여하지 않습니다.

이 메소드는 200 상태 코드와 관련 응답을 반환합니다. 이 경우에는 문자열입니다. 왜 런 일이 발생 할까요? 설명하기 위해 먼저 Nest가 응답을 조작하기 위해 두 가지 다른 옵션을 사용한다는 개념을 소개하겠습니다.

| 방식 | 설명 |
|:---- |:---- |
|Standard(recommended)| 이 내장 메소드를 사용하면 요청 핸들러가 JavaScript 객체 또는 배열을 반환할 때 자동으로 JSON으로 직렬화됩니다. 그러나 JavaScript 기본 유형(예: 문자열, 숫자, 부울)을 반환하는 경우 Nest는 직렬화를 시도하지 않고 값만 보냅니다. 이렇게 하면 응답 처리가 간단해집니다. 값만 반환하면 Nest가 나머지를 처리해줍니다.</p>또한 응답의 상태 코드는 201을 사용하는 POST 요청을 제외하고 기본적으로 항상 200을 전달합니다. 처리기 수준에서 `@HttpCode(...)` 데코레이터를 추가하여 이 동작을 쉽게 바꿀 수 있습니다. | 
|Library-specific | 메소드 핸들러 서명(예: `findAll(@Res() 응답)`)에서 `@Res()` 데코레이터를 사용하여 주입할 수 있는 라이브러리별(예: `Express`) 응답 개체를 사용할 수 있습니다. 이 접근 방식을 사용하면 해당 개체에 의해 노출된 메소드 핸들러들에 따라 제공되는 기본 응답 처리 메서드를 사용할 수 있습니다. 예를 들어 `Express`를 사용하면 `response.status(200).send()`와 같은 코드를 사용하여 응답을 구성할 수 있습니다. |

> 주의사항 
> Nest는 핸들러가 `@Res()` 또는 `@Next()`를 사용하는 시기를 감지하여 라이브러리별 옵션을 선택했음을 나타냅니다. 두 접근 방식을 동시에 사용하는 경우 이 단일 경로에 대해 표준 접근 방식이 `자동으로 비활성화`되고 더 이상 예상대로 작동하지 않습니다. 두 가지 접근 방식을 동시에 사용하려면 `@Res({ passthrough: true })`에서 passthrough 옵션을 true로 설정해야 합니다. (예: 응답 객체를 삽입하여 쿠키/헤더만 설정하고 나머지는 프레임워크에 남겨 두는 방식)

#### Request object 

핸들러는 클라이언트 요청 세부정보에 액세스해야 하는 경우가 많습니다. Nest는 기본 플랫폼(기본적으로 Express)의 요청 객체에 대한 액세스를 제공합니다. 핸들러의 서명에 `@Req()` 데코레이터를 추가하여 Nest에 요청 객체를 삽입하도록 지시하여 요청 객체에 액세스할 수 있습니다.

```typescript
// cats.controller.ts
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(@Req() request: Request): string {
    return 'This action returns all cats';
  }
}
```

> HINT 
> `express` 타이핑 방식의 이점을 취하고 싶다면, `@types/express` 패키지를 설치하세요.

요청 객체는 HTTP 요청을 나타내며 요청 쿼리 문자열, 매개변수, HTTP 헤더 및 본문에 대한 속성을 갖습니다(자세한 내용은 [여기](https://expressjs.com/en/api.html#req) 참조). 대부분의 경우 이러한 속성을 수동으로 가져올 필요는 없습니다. 대신 기본적으로 사용할 수 있는 `@Body()` 또는 `@Query()`와 같은 전용 데코레이터를 사용할 수 있습니다. 다음은 제공된 데코레이터와 이들이 나타내는 일반 플랫폼별 객체 목록입니다.

| 데코레이터 | 객체 |
|:----------|:-----|
|`@Request(), @Req()`	| `req` |
|`@Response(), @Res()*`| `res` |
|`@Next()`|`next`|
|`@Session()`|`req.session`|
|`@Param(key?: string)`|	`req.params` / `req.params[key]`|
|`@Body(key?: string)`|	`req.body` / `req.body[key]`|
|`@Query(key?: string)`|	`req.query` / `req.query[key]`|
|`@Headers(name?: string)`|	`req.headers` / `req.headers[name]`|
|`@Ip()`|	`req.ip`|
|`@HostParam()`|	`req.hosts`|

`*`기본 HTTP 플랫폼(예: Express 및 Fastify) 전반의 입력 호환성을 위해 Nest는 `@Res()` 및 `@Response()` 데코레이터를 제공합니다. `@Res()`는 단순히 `@Response()`의 별칭입니다. 둘 다 기본 네이티브 플랫폼 응답 개체 인터페이스를 직접 노출합니다. 이를 사용할 때 최대한 활용하려면 기본 라이브러리에 대한 입력(예: `@types/express`)도 가져와야 합니다. 메소드 핸들러에 `@Res()` 또는 `@Response()`를 삽입하면 Nest를 해당 핸들러에 대한 라이브러리 특정 모드로 설정하고 응답 관리를 담당하게 됩니다. 그렇게 할 때 응답 개체(예: res.json(...) 또는 res.send(...))를 호출하여 일종의 응답을 발행해야 합니다. 그렇지 않으면 HTTP 서버가 중단됩니다.

#### Resources

앞서 우리는 cats 리소스(`GET` 경로)를 가져오기 위한 엔드포인트를 정의했습니다. 또한 일반적으로 새 레코드를 생성하는 엔드포인트를 제공하려고 합니다. 이를 위해 POST 핸들러를 생성해 보겠습니다.

```typescript
import { Controller, Get, Post } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Post()
  create(): string {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
```

Nest는 `@Get()`, `@Post()`, `@Put()`, `@Delete()`, `@Patch()`, `@Options()` 및 `@Head()` 등 모든 표준 HTTP 메서드에 대한 데코레이터를 제공합니다. 또한 `@All()`은 이를 모두 처리하는 엔드포인트를 정의합니다.

#### Route wildcards

루트 기반의 패턴 방식 지원합니다. 예를 들어 애스터리스크는 와일드 카드로 사용이 가능하고, 어떤 문자의 조합도 매칭 시킬 수 있습니다. 

```typescript
@Get('ab*cd')
findAll() {
  return 'This route uses a wildcard';
}
```

`ab*cd` 루트 경로는 `abcd`, `ab_cd`,... 등등에 매칭이 가능합니다. 문자 `?`, `+`, `*`, `()` 들은 루트 경로를 위해 사용이 될 수 있다. 그리고 이런 문자들은 정규 표현식에 대응하는 하위 집합입니다. 하이픈(`-`)과 점(`.`) 의 경우 문자열 기반의 경로로 해석됩니다. 

> HINT
> 경로 중간에 있는 와일드 카드는 Express 에서만 지원됩니다. 

#### Status code

앞에 서 언급했드, response의 상태 코드는 기본적으로 200이 기본값입입니다. POST는 201입니다. 우리는 이것에 대해서 `@HttpCode(...)` 데코레이터를 활용해 행동을 수정할 수 있습니다. 

```typescript
@Post()
@HttpCode(204)
create() {
  return 'This action adds a new cat';
}
```

> HINT
> `HttpCode`를 `@nestjs/common` 패키지에서 임포트 해서 사용하십시오.

종종 상태 코드는 다양한 요소에 의해 결정됩니다. 이런 경우 당신은 라이브러리 특화 response(`@Res()`를 사용하는 걸 주입한다던가) 객체를 사용할 수도 있다. 

#### Headers

response 헤더를 커스텀하기 위해, `@Header()` 데코레이터를 사용하거나, 라이브러리 특화 response 객체를 사용하시면 됩니다. 

```typescript
@Post()
@Header('Cache-Control', 'none')
create() {
  return 'This action adds a new cat';
}
```

#### Redirection

특정 URL로 response를 리다이렉션 시키기 위해선, `@Redirect()` 데코레이터나 혹은 라이브러리들이 제공하는 객체를 활용하십시오.

`@Redirect()`는 인자 두개를 취하는데, `url`과 `statusCode` 입니다. 양쪽 다 옵셔널이며, 생략되었을 때는, 기본적으로 `statusCode`는 `302`입니다. 

```typescript
@Get()
@Redirect('https://nestjs.com', 301)
```

> HINT
> 때때로 HTTP 상태 코드 혹은 URL 리다이렉션을 동적으로 결정 하고 싶을 수 있습니다. 이럴 때는 다음 객체를 반환함으로써 구현이 가능합니다. `HttpRedirectResponse` 인터페이스(`@nestjs/common`)

반환된 값은 `@Redirect()` 데코레이터에 전달될 인자를 덮어쓰기도 가능합니다. 

```typescript
@Get('docs')
@Redirect('https://docs.nestjs.com', 302)
getDocs(@Query('version') version) {
  if (version && version === '5') {
    return { url: 'https://docs.nestjs.com/v5/' };
  }
}
```

#### Route parameters 
정적인 경로를 가진 라우트는 동적 데이터를 request의 부분으로 삼고 있는 것을 수용하길 원할 때, 정상적으로 동작하지 않을 것입니다.(e.g., `GET /cats/1`, id 1인 cat) 인자와 함께 루트를 정의 내리기 위해선, request URL 안의 위치에 동적인 값을 취하기 위해 루트의 경로 상에 패러미터 토큰들을 넣어 둘 수 있다. `@Get()` 안에 있는 인자 토큰 루트 예시는 이러한 사용 방식을 아래 예시를 통해 보여줍니다. 이 방식으로 선언된 루트 인자의 경우 `@Param()` 데코레이터를 통해 접근이 가능하며, 메서드 식별자로 추가 될 수 있다. 

> HINT
> 인자를 가진 루트는 정적 경로 후에 선언되어야 한다. 이렇게 하면 매개변수화된 경로가 정적 경로로 향하는 트래픽을 가로 채는 것을 방지할 수 있다. 

```typescript
@Get(':id')
findOne(@Param() params: any): string {
  console.log(params.id);
  return `This action returns a #${params.id} cat`;
}
```

`@Param()`은 메서드 인자로 사용되는 데코레이터 이고, 메서드의 바디 안에 메서드 인자로 데코레이트된 프로퍼티들로써 루트 패러미터를 이용 가능하게 만들어준다. 위의 코드에서 볼 수 있듯, 우리는 `id` 패러미터를 `params.id` 라는 식으로 참조하여 접근이 가능합니다. 이렇게 데코레이터를 통해 특정 인자 토큰을 전달하는게 가능하며, 그 뒤에 루트 인자로 직접 메서드 바디 안의 이름을 통해 참조가 가능하다. 

```typescript
@Get(':id')
findOne(@Param('id') id: string): string {
  return `This action returns a #${id} cat`;
}
```

#### Sub-Domain Routing
`@Controller` 데코레이터는 `host` 라는 옵션을 취할 수 있고, 이는 들어오는 요청에서 매칭 시켜야하는 특정한 값으로 HTTP host를 요구하게 만듭니다. 

```typescript
@Controller({ host: 'admin.example.com' })
export class AdminController {
  @Get()
  index(): string {
    return 'Admin page';
  }
}
```

> WARNING
> Fastify의 nested router들을 위한 부족한 지원으로 인해, 서브 도메인 라우팅을 써야 한다면, default 값인 Express adapter를 반드시 대신 사용해야 합니다. 

`path`를 라우팅 해주는 것고 유사하게, `host` 옵션은 호스트 명 안에 위치에 대한 동적 값을 취하기 위해 토큰들을 사용할 수 있다. `@Controller` 데코레이터 안에 호스트 인자 토큰은 아래에서 어떻게 사용되는지를 보여줍니다. 이러한 방식으로 선언된 호스트 인자는 `@HostParam()` 이라는 데코레이터를 사용함으로써 접근이 가능하며, 이는 메서드 시그니쳐로 반드시 추가 되어야 합니다.

```typescript
@Controller({ host: ':account.example.com' })
export class AccountController {
  @Get()
  getInfo(@HostParam('account') account: string) {
    return account;
  }
}
```

#### Scopes
다른 프로그램 언어 백그라운드를 가진 사람들에게, Nest에서 들어오는 request 전반에 걸쳐 모든 것이 공유 된다는 사실은 예상치 못한 부분일 수 있습니다. 데이터베이스에 대한 연결 풀, 전역 상태를 포함하는 싱글톤 등등... Node.js에서는 모든 요청이 별도의 스레드에 의해 처리되는 기존의 request/ response Multi-Thrread Stateless Model을 따르지 않는다는 사실을 기억해야 합니다. 따라서 싱글톤 인스턴스를 사용하는 것은 Nest에서는 완전히 안전함을 보장해줍니다. 

그러나, Controller의 요청 기반의 lifetime이 원하는 동작일 수 있는 극단적인 경우가 있습니다.(eg. GraphQL 어플리케이션의 request 별 캐싱, request 추적, 또는 Muli-tenancy이 포함됩니다.) 따라서 [여기](https://docs.nestjs.com/fundamentals/injection-scopes)에서 제어 범위를 어떻게 제어하는지를 배울 수 있습니다.  

#### Asyncronicity
우리는 모던 자바스크립트를 사랑하고, 데이터의 추출은 대부분 비동기적이라는 사실을 알 것입니다. 이게 Nest가 `async` 함수가 잘 동작하며, 지원되는 이유입니다. 

> HINT 
> `async / await` 은 [여기](https://kamilmysliwiec.com/typescript-2-1-introduction-async-await/)에서 더 자세히 배울 수 있습니다.

모든 async 함수들은 반드시 `Promise`를 반환해야 합니다. 이는 Nest가 스스로 해결하여 지연된 반환 값을 반환 받는 다는 것을 의미 합니다. 

```typescript
//cats.controller.ts
@Get()
async findAll(): Promise<any[]> {
  return [];
}
```

위의 예시 코드는 아주 적절한 유효한 코드이다. 게다가, Nest의 라우터 핸들러가 심지어 [RxJS Observable streams](https://reactivex.io/rxjs/class/es6/Observable.js~Observable.html) 를 반환할 수 있음으로써 매우 강력하다는 것을 알 수 있다. Nest는 자동으로 수면 아래에서 자원을 구독한 상태로 있고, 마지막 반환되는 값을 취한다(일단 stream이 완료되면)

```typescript
//cats.controller.ts
@Get()
findAll(): Obsrevabl<an[]> {
  return of([]);
}

```

두 접근법 모두 동작하며, 기호에 맞춰 사용하시면 됩니다. 

#### Request payloads

우리의 이전 Post 라우트 핸들러에 대한 예시에서는 client 파라미터를 수용하지 않았습니다. 이를 수정하여 `@Body()` 데코레이터를 더함으로 수정해봅시다. 

하지만 우선, 우리는 먼저 DTO(Data Transfer Object) 스키마를 결정할 필요가 있습니다. DTO는 일종의 객체로 어떻게 데이터를 네트워크를 통해 보낼 것인가에 대한 정의를 내리는 객체 입니다. 우리는 TypeScript의 인터페이스를 활용하여, DTO의 스키마를 결정하거나, 혹은 단순한 클래스를 활용합니다. 흥미롭게도 저희는 class를 사용하는 것을 추천드립니다. 왜냐하면 클래스들은 JavaScript ES6 표준에 일부이며, 그러므로 클래스는 컴파일된 JS 안에서 실제 엔티티로써 보호 받기 때문입니다. 반면에, 변환 과정에서 interface 들은 지워지므로 Nest 는 런타임 시에 해당 내용을 참조할 수 없습니다. 이는 매우 중요한데, 왜냐하면 런타임 시에 다양한 메타 데이터 타입에 접근할 때, **Pipes** 처럼 기능이 추가적인 기능성을 제공해주기 때문이다. 

우선 `CreateCatDto` 클래스를 만들어 주자.

```typescript
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

DTO는 기본적인 프로퍼티들만을 갖고 있다. 이제 `CatsController` 내에서 새롭게 DTO를 사용할 수 있다. 

> HINT 
> 우리의 `ValidationPipe` 는 메서드 핸들러에 의해 받아지지 못하는 프로퍼티들을 필터링할 수 있다. 이런 경우, 우리는 수용 가능한 프로퍼티들에 대한 화이트리스트를 지정할 수 있고, 화이트 리스트에 포함되지 않은 어떤 프로퍼티들은 결과적으로 객체를 반환하는 것으로부터 자동으로 제거될 수 있다. `CreateCatDto` 예제 상에서, 화이트리스트라 함은, name, age, breed 프로퍼티들이다. 자세한 내용은 [여기](https://docs.nestjs.com/techniques/validation#stripping-properties)를 참고할 것 

#### Handling errors 

error 처리에 대한 세부 챕터를 참고할 것([여기](https://docs.nestjs.com/controllers#scopes))

#### Full resource sample

기본 컨트롤러의 예시 코드는 다음과 같다. 

```typescript 
import { Controller, Get, Query, Post, Body, Put, Param, Delete } from '@nestjs/common';
import { CreateCatDto, UpdateCatDto, ListAllEntities } from './dto';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(@Query() query: ListAllEntities) {
    return `This action returns all cats (limit: ${query.limit} items)`;
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return `This action returns a #${id} cat`;
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() updateCatDto: UpdateCatDto) {
    return `This action updates a #${id} cat`;
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return `This action removes a #${id} cat`;
  }
}
```

#### Getting up and running 

#### Libaray-specific approach 

