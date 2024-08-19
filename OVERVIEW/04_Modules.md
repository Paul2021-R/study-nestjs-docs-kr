### Modules
`@Module()` 데코레이터를 어노테이션으로 달고 있는 클래스를 모듈이라고 부릅니다. `@Module()` 데코레이터는 Nest 가 어플리케이션의 구조를 조직화하는데 사용하는 메타 데이터들을 제공해줍니다.

![](../src/04_01.png)

각 어플리케이션은 최소 하나의 모듈을 가지고 있고, 이를 `root` 모듈이라고 부릅니다. 루트모듈은 **어플리케이션 그래프** 를 생성하기위해 Nest가 사용하는 스타팅 포인트 역할을 합니다. application graph 라는 것은 Nest에서 모듈과 프로바이더 사이의 관련성, 의존성을 해결하기 위해 사용하는 것입니다. 이론적으로 작은 어플리케이션이라면 루트 모듈만 존재할 수도 있지만, 보통 일반적인 경우 그렇지 않습니다. 또한 Nest 에서는 당신의 컴포넌트 들을 조직화 할 때 효과적인 방법으로 가장 강력히 추천하는 것이 모듈형태로 사용하는 것을 강조하고 싶습니다. 따라서 대부분의 애플리케이션에서 결과 아키텍처는 여러 모듈을 사용하며 각 모듈은 밀접하게 관련된 기능 세트를 캡슐화하는 구조가 될 것입니다. 

`@Module()` 데코레이터는 모듈을 묘사하기 위한 프로퍼티를 가진 단일 객체를 취합니다. 

| 이름 | 설명 |
|:---- | :--- |
|providers|프로바이더는 Nest 의 주입기에 의해 인스턴스화 하며, 특정 모듈을 최소 넘어서서 공유될 것입니다. |
|controllers|특정 모듈에 정의 내려진 일련의 컨트롤러들은 반드시 인스턴스화 되어야 합니다.| 
|imports|해당 모듈 안에서 필요로 되어지는 프로바이더들을 export하는 모듈들의 리스트입니다.|
|exports|프로바이더들의 하위 셋으로, 이는 특정 이 모듈에 의해 제공되는 기능들로 볼 수 있으며, 이는 다른 모듈에 의해 이용이 가능한 일련의 프로바이더들을 담고 있습니다. 당신은 여기서도 프로바이더 자체를 혹은 단순히 `provide` 값의 토큰 만을 제공할 수도 있습니다.|

모들은 기본적으로 프로바이더들을 '캡슐화'합니다. 이는 현재 모듈의 직접 일부도 아니고 가져온 모듈에서 내보내지지도 않은 공급자를 주입하는 것이 불가능하다는 것을 의미합니다. 따라서 모듈에서 내보낸 공급자(exported Providers)를 모듈의 공용 인터페이스 또는 API로 간주할 수 있습니다

### Features modules 
`CatscController` 와 `CatsService`는 동일한 어플리케이션 도메인에 속합니다. 그들이 밀접하게 관련성을 가진 것 처럼, 이들을 feature module 로 이동시키는 것도 가능합니다. `feature module` 은 특정 기능, 조직화된 코드를 유지하고, 명확한 경계를 만드는데 사용됩니다. 이는 우리가 복잡함을 관리하고, 팀이 성장 혹은 어플리케이션의 사이즈가 성장할 때 특히 [SOLID]()원칙을 지키며 개발하는 것을 도와줍니다. 

위의 설명대로 시연하고자, `CatsModule` 을 생성해봅시다. 
```typescript
// CatsModule
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

> HINT 
> CLI로 모듈을 생성하기 위해선 다음과 같이 명령어를 치면 됩니다. `> nest g module cats`

위처럼, 우리는 `cats.module.ts` 안에 `CatasModule` 을 정의내렸습니다. 그리고 `cats`디렉토리에 이 모듈과 연관이 있는 모든 곳으로 옮겨 다닐 수 있게 되었습니다. 이제 마지막 남은 우리의 일은 root 모듈이 이 모듈을 import 하는 것입니다.(`AppModule` 로)
```typescript
// app.module.ts 
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule {}
```
이제 전체 구조는 다음처럼 보이게 됩니다. 

![alt text](../src/04_02.png)

### Shared modules
Nest는 기본적으로 싱글톤 디자인으로 모듈이 구성되어 있습니다. 그래서 만약 목수의 모듈들에서 효율적으로 어떤 프로바이더의 인스턴스를 같이 공유하고, 사용할 수 있게 되입니다. 

![](../src/04_03.png)()

모든 모듈들은 자동적으로 **shared module**이 됩니다. 일단 어떤 모듈이든지 간에 생성된 이래 재사용이 가능해집니다. 

```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
export class CatsModule {}
```

예시 코드를 보면, 이제 `CatsService`는 어떤 곳에서든지 사용이 가능한 공유 모듈이 되었습니다. `CatsModule`을 import 해줌으로써, 어떤 모듈이든 서비스를 사용 가능합니다. 

### Module re-exporting
상단에서 본 것처럼, 모듈들은 내부 프로바이더들을 export 하는 것이 가능합니다. 게다가, 모듈들은 모듈들이 import 한 것을 재 export 하는 것도 가능합니다. 아래의 예시를 보면 `CommonModule`는 `CoreModule` 기준으로 imported 되고, 동시 exported 된 것을 볼 수 있다. 이렇게 함으로써 다른 모듈들은 다시 `CommonModule`을 CoreModule 에서 import 받아서 사용이 가능한 것이다. 

```typescript
@Module({
  imports: [CommonModule],
  exports: [CommonModule],
})
export class CoreModule {}
```

### Dependency injection 

모듈은 기본적으로 프로바이더들로 주입이 가능하다. 

```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {
  constructor(private catsService: CatsService) {}
}
```

그러나, 모듈 클래스 자체는 순환 의존성으로 인해 공급자로 주입될 수 없다.

### Global modules 

만약 어디서든지 같은 모듈의 셋을 import 할 필요가 있을 때, 이는 매우 지루한 일이 될 것이다. Nest 와 다르게 Angular 의 프로바이더들은 전역으로 등록이 됩니다. 일단 정의 내리면, 어디서든지 사용이 가능합니다. 하지만 네스트의 경우, 모듈의 범위 안에서 프로바이더들을 캡슐화 합니다. 따라서 캡술화된 모듈 안에 importing태없이는 어디서든지 모듈의 프로바이더들을 사용할 수가 없습니다. 

즉시 사용 가능한 모든 곳에서 사용할 수 있는 프로바이더의 세트를 제공하고 싶다면, 모듈에 `@Global()` 이라고 불리는 데코레이터를 붙여서, 전역으로 만들면 됩니다. 

```typescript
import { Module, Global } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Global()
@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```

`@Global()` 데코레이터는 모듈을 global-scope 로 만들어 줍니다. 글로벌 모듈은 오직 한번 등록이 되어야 하며, 일반적으로 루트 혹은 코어 모듈에 의해 지정됩니다. 위의 예시처럼, `CatsService` 프로바이더는 이제 단일하게 사용이 가능하며, 서비스를 주입하려는 모듈은 imports 배열에서 CatsModule을 가져올 필요가 없습니다. 

> HINT 
> 모든 것들을 글로벌로 만드는 것은 좋은 디자인 결정이라고 볼순 없습니다. 글로벌 모듈은 필요한 상용구의 양을 줄일 수 있습니다. imports 배열이 일반적으로 소비자가 모듈의 API를 사용할 수 있도록 하는 데 선호 되는 방법이다. 

### Dynamic moudles 

네스트 모듈 시스템은 **다이나믹 모듈** 이라고 불리는 강력한 기능을 포함하고 있습니다. 이 기능은 프로바이더들을 등록하거나 설정을 동적으로 가능하게 하는 커스터마이저블한 모듈을 쉽게 생성하는 것을 가능케 만들어줍니다. 다이나믹 모듈에 대해 보다 자세한 내용은 [여기](https://docs.nestjs.com/fundamentals/dynamic-modules)를 확인해주세요. 이 챕터에서는 간단하게 모듈에해대한 소개를 함으로 마무리 지어보고자 합니다. 

다음 예시는 `DatabaseModule`을 위한 다이나믹 모듈 정의의 예시르 보여줍니다. 

```typescript
import { Module, DynamicModule } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
  providers: [Connection],
  exports: [Connection],
})
export class DatabaseModule {
  static forRoot(entities = [], options?): DynamicModule {
    const providers = createDatabaseProviders(options, entities);
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}
```

> HINT 
> `forRoot()` 메서드는 다이나믹 모듈을 동시기 혹은 비동기식으로 반환해줍니다. 

이 모듈은 `Connection` 프로바이더를 기본적으로 정의 내리고 있습니다. (`@Module()` 데코레이터 메터데이터 안에), 그러나 추가적으로 `forRoot()` 메서드에 전달된 엔티티와 옵션 객체에 따라 - 예를 들어 저장소와 같은 프로바이더 컬렉션을 노출합니다. 기억할 것은, 다이나믹 모듈에 의해 반환된 프로퍼티들은 `@Module()` 데코레이터 안에 정의된 베이스 모듈 메타데이터를 확장한다는 점입니다. 그것이 정적으로 선언된 `Connection` 프로바이더와 동적으로 생성된 저장소 프로바이더가 모듈에서 내보내는 방법 입니다. 

만약 전역의 범위로 다이나믹 모듈을 지정하고 싶다면, `global` 프로퍼티를 `true`로 설정하시면 됩니다. 

```typescript
{
  global: true,
  module: DatabaseModule,
  providers: providers,
  exports: providers,
}
```

> WARNING 
> 앞서 언급 했듯이, 모든 것을 전역으로 만드는 것은 좋은 디자인 결정은 아닙니다. 

`DatabaseModule`은 이제 다음 방식으로 import, configure 됩니다 

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
})
export class AppModule {}
```

만약 다이나믹 모듈을 다시 export 를 하고 싶다면, `forRoot()` 메서드를 생략하고, exports 배열 안에 지정하면 됩니다. 

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
  exports: [DatabaseModule],
})
export class AppModule {}
```

다이나믹 모듈 챕터에서 상세한 내용들을 다룰 예정이며, 실제 동작 예시도 보게 될 비입니다. 

> HINT
> Learn how to build highly customizable dynamic modules with the use of ConfigurableModuleBuilder here in this chapter.