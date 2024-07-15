### Providers

프로바이더는 Nest에 존재하는 근본적인 개념이다. 많은 기본 Nest 클래스들은 프로바이더(서비스, 레포지터리, 팩토리, helper, 등등) 칭해진다. 프로바이더라는 내겸의 핵심 철학은 의존성으로 `주입 될 수 있다` 는 점이다. 이는 객체는 다른 객체들과 서로 다양한 관계를 생성할 수 있고, 이러한 객체들의 '연결'의 기능들은 주로 Nest 런타임 시스템에 위임될 수 있습니다.  

![](../src/03_01.png)

이전 챕터에서, 우리는 간간한 `CatsController`를 만들었습니다. 컨트롤러들은 HTTP 요청을 처리해야 하고, 더 복잡한 작업에 대해선 `Providers`에게 위임합니다. 프로바이더들은 순수한 JavaScript 클래스들이며, 이들을 모듈 안에서 `Providers` 로 선언합니다. 

> HINT
> Nest 가 좀더 객체 지향적 방법으로 의존성을 디자인하고, 구조화 하는게 가능해진 이래로, 우리는 SOLID 원칙을 따르기를 강력히 권장 합니다. 

### Services

이제 간단한 `CatsService` 를 생성해봅시다. 이 서비스는 데이터 저장, 재 탐색, 그리고 `CatsController`에 의해 사용될 목적으로 디자인 되었습니다. 그렇기에 프로바이더로 정의되기에 훌륭한 후보 입니다. 

```typescript
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
```

> HINT 
> CLI 상에서 서비스를 생성하기 위해서, 간단히 `$ nest g service cats` 라고 입력하면 됩니다. 

생성한 서비스는 하나의 프로퍼티를 가지며, 두개의 메서드를 가진 기본적인 클래스입니다. 여기서 한가지 새로운 기능으로는 `@Injectable()` 이라는 데코레이터가 사용되었다는 점입니다. 이 데코레이터는 메타 데이터를 붙이는 역할을 하며, `CatsService` 가 Nest의 IoC 컨케이너에 의해 관리 될 수 있는 클래스라는 점을 선언하는 것입니다. 

```typescript
export interface Cat {
  name: string;
  age: number;
  breed: string;
}
```

서비스에 사용될 타입도 만들었으니, 이제는 서비스 클래스를 통해 고양이들을 탐색해볼 차례입니다. 

```typescript
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

`CatsService` 는 클래스 생성자를 통해 주입되었다. `private` 문법의 용례를 주의깊게 보세요. 이 짧은 문구를 통해 우리는 해당 프로바이더 멤버를 선언도 하고, 초기화도 즉시, 동일한 장소에서 해결할 수 있었습니다. 

### Dependency Injection

Nest는 Dependency Injection이라고 불리는 대중적이며 아주 강력한 디자인 패턴을 기반으로 만들어졌습니다. 우리는 공식 Angular 문서가 해당 개념에 관한 좋은 기사이므로, 읽어보길 권장 드립니다. 

Nest 상에서는, TS의 호환성 덕분에, 의존성을 관리하는 것이 매우 쉽게 구현되어 있습니다. 왜냐하면 type에 의해 해결이 용이하기 때문입니다. 하단 예시에서, Nest 는 `catService` 를 `CatsServuce`의 인스턴스를 생성 및 반환함으로써 의존성 문제를 해결한다. (혹은, 싱글톤 패턴의 일반적인 상황 하에서는, 요청된 어딘가에서 이미 존재하는 인스턴스를 반환한다.) 이 의존성이란 것은 당신의 컨트롫러의 생성자로 해결되며, 전달된다(혹은 지어된 프로퍼티로 할당된다.)

```typescript 
constructor(private catsService: CatsService) {}
```

### Scopes

프로바이더는 일반적으로 application의 라이프사이클과 동기화된 라이프 타임(scope)를 갖고 있습니다. application이 부트스트랩될 때, 모든 의존성은 반드시 resolve 되어야 하며, 그러므로 프로바이더들은 인스턴스화 되어야 합니다. 이와 함께, application이 종료 될 때, 각 프로바이더 역시 destroy 되어야 합니다. 그러나 공급자 수명 요청 범위를 지정하는 방법도 있고, 이 기술에 대해선 [여기](https://docs.nestjs.com/fundamentals/injection-scopes)를 참고하십시오.

### Custom providers
Nest는 IoC(제어의 역전) 컨테이너를 빌트인하고 있고, 이는 프로바이더들 사이의 관계성을 resolve 해줍니다. 이 기능은 위에서 언급한 의존성 주입의 기능의 기반이 된다. 하지만 실제론 우리가 언급하는 것 이상으로 더 강력합니다. 프로바이더를 정의내리는 방법은 몇가지 방법이 존재합니다. :
순수한 값, 클래스, 그리고 비동기 혹은 동기의 factory를 활용할 수 있으며 더 많은 예시는 [여기](https://docs.nestjs.com/fundamentals/custom-providers)를 참고 하세요.

### Optional providers
때때로, resolve 할 필요가 없는 의존성들도 존재합니다. 예를 들어, 당신이 제작한 클래스는 `configuration object`에 의존합니다. 그러나 none 이 전달된다면, 기본값이 전달될 것입니다. 이러한 경우에 따라, 의존성은 선택적이 될 수 있고, configuration 프로바이더의 부족은 에러를 야기하지 않을 수 있습니다. 

프로바이더를 옵셔널로 가리키기 위해, `@Optional()` 데코레이터를 생성자의 시그니쳐에 달아 줌으로써 사용도 가능하다. 

```typescript 

import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  constructor(@Optional() @Inject('HTTP_OPTIONS') private httpClient: T) {}
}
```
위의 예시를 보면, 우리는 커스텀 프롭이더를 사용하고 있습니다. 이는 우리가 `HTTP_OPTIONS`라는 커스텀 **token**을 포함하고 있기 때문입니다. 이전 예시가 생성자 기반으로 생성자 안에서 클래스를 통해 의존성을 가리키는 주입을 보여주었습니다.커스텀 프로바이더에 대해 더 읽고 싶거나, token들에 대해 보고 싶다면 [여기](https://docs.nestjs.com/fundamentals/custom-providers)를 읽어 보세요.

### Property-based injection
지금까지 사용했던 방식의 기술은 생성자 기반의 주입이라고 불리며, 프로바이더들은 생성자 메서드를 경유해서 주입되는 프로바이더들입니다. 특수한 케이스로, Property-based injection 가 유용한 경우도 존재합니다. 예를 들어, 만약 top-level 클래스가 다수의 프로바이더 혹은 하나의 프로바이더에 의존하고 있다면, 생성자의 하위 클래스에서 `super()`를 호출하는 방법으로 공급자를 끝까지 전달하는 것은 매우 지루할 수 있습니다. 이를 피하기 위해, `@Inject()` 데코레이터를 프로퍼티 래벨에서 사용할 수 있습니다. 

```typescript
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  @Inject('HTTP_OPTIONS')
  private readonly httpClient: T;
}
```
> WARNING 
> 만약 당신의 클래스가 다른 클래스로 확장하지 않는다면, 당신은 아마도 constructor-based 주입을 사용하는 것을 선호할 것이다. 생성자는 요청되는 의존성이 무엇인지, 그리고 `@Inject`와 함께 어노테이션되는 클래스 속성보다 더 좋은 가독성을 제공해준다. 

### Provider registration
이제 프로바이더를 정의 내릴수 있습니다(`CatsService`). 그리고 이제 서비스의 소비자를 가질수 있으며, Nest와 함께 서비스를 등록할 필요가 있고, 그 결과 주입을 수행할 수 있습니다. 

### Manual instantiation
