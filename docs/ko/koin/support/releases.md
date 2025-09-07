---
title: 릴리스 및 API 업그레이드 가이드
custom_edit_url: null
---

:::info
이 페이지는 Koin의 모든 주요 릴리스에 대한 포괄적인 개요를 제공하며, 프레임워크의 진화를 자세히 설명하여 업그레이드를 계획하고 호환성을 유지하는 데 도움을 줍니다.
:::

각 버전에 대해 문서는 다음 섹션으로 구성됩니다.

- `Kotlin`: 릴리스에 사용된 Kotlin 버전을 명시하여 언어 호환성을 명확히 하고 최신 Kotlin 기능을 활용할 수 있도록 합니다.
- `New`: 기능성과 개발자 경험을 향상시키는 새롭게 도입된 기능과 개선 사항을 강조합니다.
- `Experimental`: 실험적(experimental)으로 표시된 API 및 기능을 나열합니다. 이들은 활발히 개발 중이며 커뮤니티 피드백에 따라 변경될 수 있습니다.
- `Deprecated`: 더 이상 사용되지 않음(deprecated)으로 표시된 API 및 기능을 식별하며, 권장 대안에 대한 지침을 제공하여 향후 제거에 대비할 수 있도록 돕습니다.
- `Breaking`: 하위 호환성(backward compatibility)을 깨뜨릴 수 있는 변경 사항을 상세히 설명하여 마이그레이션 중 필요한 조정 사항을 인지하도록 합니다.

이러한 구조화된 접근 방식은 각 릴리스의 점진적인 변경 사항을 명확히 할 뿐만 아니라 Koin 프로젝트의 투명성, 안정성, 지속적인 개선에 대한 우리의 약속을 강화합니다.

자세한 내용은 [API 안정성 계약](api-stability.md)을 참조하세요.

## 4.1.1

:::note
Kotlin `2.1.21` 버전을 사용합니다.
:::

### New 🎉

`koin-compose-viewmodel-navigation`
- Compose 내비게이션 지원 강화를 위해 선택적 `navGraphRoute` 매개변수가 추가된 `sharedKoinViewModel` 개선

`koin-core`
- 코어 리졸버 성능 최적화 - 단일 스코프 확인(resolution)으로 불필요한 평탄화 방지
- 연결된 스코프 ID 표시로 스코프 디버깅 개선

### Library Updates 📚

- **Kotlin** 2.1.21 (from 2.1.20)
- **Ktor** 3.2.3 (from 3.1.3)
- **Jetbrains Compose** 1.8.2 (from 1.8.0)
- **AndroidX**: Fragment 1.8.9, WorkManager 2.10.3, Lifecycle 2.9.3, Navigation 2.9.3
- **Testing**: Robolectric 4.15.1, Benchmark 0.4.14
- **Build**: Binary Validator 0.18.1, NMCP 1.1.0

### Bug Fixes 🐛

`koin-core`
- 호환성 오류를 유발하던 로거 제약 조건 롤백
- `LocalKoinApplication`/`LocalKoinScope` 컨텍스트 처리 개선으로 Compose 스코프 확인(resolution) 수정

`koin-build`
- Maven Central 게시 문제 수정

## 4.1.0

:::note
Kotlin `2.1.20` 버전을 사용합니다.
:::

### New 🎉

`koin-core`
- 설정 - 설정을 래핑하는 데 도움이 되는 `KoinConfiguration` API
- 스코프 - 스코프 범주를 위한 전용 스코프 타입 한정자(*Scope Archetype*)의 새로운 *스코프 아키타입*을 도입합니다. 이제 스코프 범주(일명 아키타입)에 대해 인스턴스 확인(resolution)을 수행할 수 있습니다.
- 기능 옵션 - Koin 내에서 새로운 기능 동작을 기능 플래그(feature flag)하는 데 도움이 되는 "기능 옵션"입니다. Koin 설정에서 `options` 블록을 사용하여 옵션을 활성화할 수 있습니다:
```kotlin
startKoin {
    options(
        // activate a new feature
        viewModelScopeFactory()
    )
}
```
- 코어 - `ResolutionExtension`을 통해 Koin이 외부 시스템이나 리소스에서 확인(resolve)하도록 돕는 새로운 `CoreResolver`를 도입합니다 (Ktor DI를 연결하는 데 사용됩니다).

`koin-android`
- 업그레이드된 라이브러리(`androidx.appcompat:appcompat:1.7.0`, `androidx.activity:activity-ktx:1.10.1`)는 최소 SDK 레벨을 14에서 21로 높여야 합니다.
- DSL - Activity/Fragment 내에 스코프를 선언하기 위한 새로운 Koin 모듈 DSL 확장 `activityScope`, `activityRetainedScope`, `fragmentScope`가 추가되었습니다.
- 스코프 함수 - 또한 `activityScope()`, `activityRetainedScope()` 및 `fragmentScope()` API 함수는 이제 스코프 아키타입을 트리거합니다.

`koin-androidx-compose`
- Koin Compose Multiplatform 및 모든 Compose 1.8 & Lifecycle 2.9에 맞춰 조정되었습니다.

`koin-compose`
- Compose 1.8 & Lifecycle 2.9에 맞춰 조정되었습니다.
- 새로운 함수 - Android Studio 및 IntelliJ에서 병렬 미리보기를 렌더링하는 데 도움이 되는 `KoinApplicationPreview`

`koin-compose-viewmodel`
- 상위 Activity를 호스트로 설정할 수 있도록 `koinActivityViewModel`이 추가되었습니다.

`koin-ktor`
- 멀티플랫폼 - 이제 이 모듈은 Kotlin KMP 형식으로 컴파일됩니다. 멀티플랫폼 프로젝트에서 `koin-ktor`을 타겟팅할 수 있습니다.
- 병합 - 이전 koin-ktor3 모듈이 koin-ktor로 병합되었습니다.
- 확장 - Ktor 모듈에 직접 결합된 Koin 모듈을 선언할 수 있도록 `Application.koinModule { }` 및 `Application.koinModules()`를 도입합니다.
```kotlin
fun Application.customerDataModule() {
    koinModule {
        singleOf(::CustomerRepositoryImpl) bind CustomerRepository::class
    }
}
```
- 스코프 - `Module.requestScope` - Ktor 요청 스코프 내에서 정의를 선언할 수 있도록 합니다 (`scope<RequestScope>`를 수동으로 선언하는 것을 피합니다).
주입된 스코프는 또한 생성자에서 `ApplicationCall`을 주입할 수 있도록 합니다.

`koin-core-coroutines`
- 모듈 DSL - 모듈 설정을 하나의 구조로 모으는 데 도움이 되는 새로운 `ModuleConfiguration`을 도입하여 나중에 더 잘 확인할 수 있도록 합니다.
```kotlin
val m1 = module {
    single { Simple.ComponentA() }
}
val lm1 = lazyModule {
    single { Simple.ComponentB(get()) }
}
val conf = moduleConfiguration {
    modules(m1)
    lazyModules(lm1)
}
```
- 설정 DSL - Koin 설정은 이제 `ModuleConfiguration`을 사용하여 모듈을 로드할 수 있습니다:
```kotlin
startKoin {
    moduleConfiguration {
        modules(m1)
        lazyModules(lm1)
    }
}

// or even
val conf = moduleConfiguration {
    modules(m1)
    lazyModules(lm1)
}

startKoin {
    moduleConfiguration(conf)
}
```

`koin-test-coroutines`
- 새로운 코루틴 관련 테스트 API를 도입하기 위해 새로운 `koin-test-coroutines` Koin 모듈이 추가되었습니다.
- 확장 - `Verify` API를 확장하여 `moduleConfiguration`으로 Koin 설정을 확인하고, Modules/Lazy Modules 혼합 구성을 확인할 수 있도록 합니다:
```kotlin
val conf = moduleConfiguration {
    modules(m1)
    lazyModules(lm1)
}

conf.verify()

// if you want Android types (koin-android-test)
conf.verify(extraTypes = androidTypes)
```

`koin-core-annotations`
- 어노테이션 - `@InjectedParam` 또는 `@Provided`는 속성을 주입된 매개변수 또는 동적으로 제공되는 것으로 간주하도록 태그(tag)합니다. 현재는 `Verify` API에서 사용되지만, 나중에 더 가벼운 DSL 선언에 도움이 될 수 있습니다.

### Experimental 🚧

`koin-core`
- Wasm - Kotlin 2.1.20 UUID 생성 사용

`koin-core-viewmodel`
- DSL - ViewModel 스코프 아키타입에 스코프가 지정된 컴포넌트를 선언하기 위한 모듈 DSL 확장 `viewModelScope`가 추가되었습니다.
- 스코프 함수 - ViewModel을 위한 스코프를 생성하는 함수 `viewModelScope()`가 추가되었습니다 (ViewModel 클래스에 연결됨). 이 API는 이제 `ViewModelScopeAutoCloseable`을 사용하여 `AutoCloseable` API를 활용하여 스코프를 선언하고 닫는 데 도움이 됩니다. 더 이상 수동으로 ViewModel 스코프를 닫을 필요가 없습니다.
- 클래스 - 즉시 사용 가능한 ViewModel 스코프 클래스(스코프 생성 및 닫기 처리)를 지원하기 위해 `ScopeViewModel` 클래스가 업데이트되었습니다.
- 기능 옵션 - ViewModel 스코프를 사용한 생성자 ViewModel 주입은 Koin 옵션 `viewModelScopeFactory` 활성화를 필요로 합니다:
```kotlin
startKoin {
    options(
        // activate a new ViewModel scope creation
        viewModelScopeFactory()
    )
}

// will inject Session from MyScopeViewModel's scope
class MyScopeViewModel(val session: Session) : ViewModel()

module {
    viewModelOf(::MyScopeViewModel)
    viewModelScope {
        scopedOf(::Session)
    }
}
```

`koin-compose`
- Compose 함수 - 멀티플랫폼 Compose 진입점을 제안하기 위해 새로운 `KoinMultiplatformApplication` 함수가 추가되었습니다.

`koin-core-viewmodel-navigation`
- 내비게이션 확장 - 내비게이션의 `NavbackEntry`에서 ViewModel 인스턴스를 재사용하기 위해 `sharedViewModel`이 추가되었습니다.

`koin-test`
- 어노테이션 - Koin 설정 검증 API `Verify`는 이제 널러블(nullable), 지연(lazy), 리스트 매개변수를 확인하는 데 도움이 됩니다. `@InjectedParam` 또는 `@Provided`를 사용하여 속성을 주입된 매개변수 또는 동적으로 제공되는 것으로 간주하도록 태그(tag)하기만 하면 됩니다. 이렇게 하면 `Verify` API에서 복잡한 선언을 피할 수 있습니다.
```kotlin
// now detected in Verify
class ComponentB(val a: ComponentA? = null)
class ComponentBParam(@InjectedParam val a: ComponentA)
class ComponentBProvided(@Provided val a: ComponentA)
```

### Deprecation ⚠️

`koin-android`
- `ScopeViewModel`은 이제 `koin-core-viewmodel`의 `ScopeViewModel` 클래스 대신 사용되도록 더 이상 사용되지 않습니다.

`koin-compose`
- Koin 컨텍스트가 현재 기본 컨텍스트에서 적절히 준비되므로 Compose 컨텍스트 API는 더 이상 필요하지 않습니다. 다음은 더 이상 사용되지 않으며 제거될 수 있습니다: `KoinContext`

`koin-androidx-compose`
- Koin 컨텍스트가 현재 기본 컨텍스트에서 적절히 준비되므로 Jetpack Compose 컨텍스트 API는 더 이상 필요하지 않습니다. 다음은 더 이상 사용되지 않으며 제거될 수 있습니다: `KoinAndroidContext`

`koin-androidx-compose-navigation`
- 라이프사이클 라이브러리 업데이트로 인해 `koinNavViewModel` 함수는 더 이상 필요하지 않으며, `koinViewModel`로 대체될 수 있습니다.

`koin-core-viewmodel-navigation`
- 라이프사이클 라이브러리 업데이트로 인해 `koinNavViewModel` 함수는 더 이상 필요하지 않으며, `koinViewModel`로 대체될 수 있습니다.

`koin-ktor`
- 확장 - `Application.koin`은 이제 `Application.koinModules` 및 `Application.koinModule`를 선호하여 더 이상 사용되지 않습니다.

### Breaking 💥

`koin-android`
- 모든 이전 상태 ViewModel API는 이제 제거되었습니다:
    - `stateViewModel()`,`getStateViewModel()`, 대신 `viewModel()`을 사용하세요.
    - `getSharedStateViewModel()`, `sharedStateViewModel()`, 공유 인스턴스를 위해 대신 `viewModel()` 또는 `activityViewModel()`을 사용하세요.

`koin-compose`
- 이전 Compose API 함수가 제거되었습니다:
    - 함수 `inject()`는 `koinInject()`를 선호하여 제거되었습니다.
    - 함수 `getViewModel()`은 `koinViewModel()`을 선호하여 제거되었습니다.
    - 함수 `rememberKoinInject()`는 `koinInject()`로 이동되었습니다.
- 함수 `rememberKoinApplication`은 `@KoinInternalAPI`로 표시됩니다.

## 4.0.4

:::note
Kotlin `2.0.21` 버전을 사용합니다.
:::

사용된 모든 라이브러리 버전은 [libs.versions.toml](https://github.com/InsertKoinIO/koin/blob/main/projects/gradle/libs.versions.toml)에서 확인할 수 있습니다.

### New 🎉

`koin-core`
- `KoinPlatformTools.generateId()` - 이 새로운 Kotlin 버전과 함께, 새로운 `kotlin.uuid.uuid` API의 이점을 얻게 됩니다. `KoinPlatformTools.generateId()` Koin 함수는 이제 이 새로운 API를 사용하여 모든 플랫폼에서 실제 UUID를 생성합니다.

`koin-viewmodel`
- Koin 4.0은 Google/Jetbrains KMP API를 통합하는 새로운 ViewModel DSL 및 API를 도입합니다. 코드베이스 전반의 중복을 피하기 위해, ViewModel API는 이제 `koin-core-viewmodel` 및 `koin-core-viewmodel-navigation` 프로젝트에 위치합니다.
- ViewModel DSL을 위한 임포트(import)는 `org.koin.core.module.dsl.*`입니다.

주어진 프로젝트의 다음 API들은 이제 안정화되었습니다.

`koin-core-coroutines` - 모든 API는 이제 안정화되었습니다.
- 모든 `lazyModules`
- `awaitAllStartJobs`, `onKoinStarted`, `isAllStartedJobsDone`
- `waitAllStartJobs`, `runOnKoinStarted`
- `KoinApplication.coroutinesEngine`
- `Module.includes(lazy)`
- `lazyModule()`
- `KoinPlatformCoroutinesTools`

### Experimental 🚧

`koin-test`
- `ParameterTypeInjection` - `Verify` API를 위한 동적 매개변수 주입 설계를 돕는 새로운 API

`koin-androidx-startup`
- `koin-androidx-startup` - `androidx.startup.Initializer` API를 사용하여 `AndroidX Startup`으로 Koin을 시작하는 새로운 기능. `koin-androidx-startup` 내의 모든 API는 실험적(Experimental)입니다.

`koin-compose`
- `rememberKoinModules` - @Composable 컴포넌트에 따라 Koin 모듈 로드/언로드
- `rememberKoinScope` - @Composable 컴포넌트에 따라 Koin 스코프 로드/언로드
- `KoinScope` - 모든 하위 Composable 자식들을 위해 Koin 스코프 로드

### Deprecation ⚠️

다음 API는 더 이상 사용되지 않음(deprecated)으로 지정되었으며, 더 이상 사용해서는 안 됩니다.

- `koin-test`
    - `checkModules`의 모든 API. `Verify` API로 마이그레이션하세요.

- `koin-android`
    - `koin-core`의 새로운 중앙 집중식 DSL을 선호하여 ViewModel DSL
    - 모든 상태 ViewModel API는 오류 수준에서 더 이상 사용되지 않습니다:
        - `stateViewModel()`,`getStateViewModel()`, 대신 `viewModel()`을 사용하세요.
        - `getSharedStateViewModel()`, `sharedStateViewModel()`, 대신 공유 인스턴스를 위해 `viewModel()` 또는 `activityViewModel()`을 사용하세요.

`koin-compose`
- 오래된 Compose API 함수는 오류 수준에서 더 이상 사용되지 않습니다:
    - 함수 `inject()`는 `koinInject()`를 선호하여 더 이상 사용되지 않습니다(오류 수준).
    - 함수 `getViewModel()`은 `koinViewModel()`을 선호하여 더 이상 사용되지 않습니다(오류 수준).
    - 함수 `rememberKoinInject()`는 `koinInject()`를 선호하여 더 이상 사용되지 않습니다(오류 수준).

- `koin-compose-viewmodel`
    - `koin-core`의 새로운 중앙 집중식 DSL을 선호하여 ViewModel DSL
    - 함수 `koinNavViewModel`은 이제 `koinViewModel`을 선호하여 더 이상 사용되지 않습니다.

### Breaking 💥

다음 API는 지난 마일스톤에서의 더 이상 사용되지 않음(deprecation)으로 인해 제거되었습니다.

:::note
`@KoinReflectAPI`로 주석 처리된 모든 API가 제거되었습니다.
:::

`koin-core`
- `ApplicationAlreadyStartedException`이 `KoinApplicationAlreadyStartedException`으로 이름이 변경되었습니다.
- `KoinScopeComponent.closeScope()`가 제거되었습니다. 내부적으로 더 이상 사용되지 않으므로
- 내부 `ResolutionContext`를 `InstanceContext`를 대체하도록 이동했습니다.
- `KoinPlatformTimeTools`, `Timer`, `measureDuration`이 제거되었으며, 대신 Kotlin Time API를 사용합니다.
- `KoinContextHandler`가 `GlobalContext`를 선호하여 제거되었습니다.

`koin-android`
- 함수 `fun Fragment.createScope()`가 제거되었습니다.
- ViewModel 팩토리(주로 내부)와 관련된 모든 API가 새로운 내부 구조를 위해 재작업되었습니다.

`koin-compose`
- 내부에서 더 이상 사용되지 않으므로 `StableParametersDefinition`이 제거되었습니다.
- 모든 Lazy ViewModel API(오래된 `viewModel()`)가 제거되었습니다.
- 내부적으로 더 이상 사용되지 않으므로 `rememberStableParametersDefinition()`이 제거되었습니다.

## 3.5.6

:::note
Kotlin `1.9.22` 버전을 사용합니다.
:::

사용된 모든 라이브러리 버전은 [libs.versions.toml](https://github.com/InsertKoinIO/koin/blob/3.5.6/projects/gradle/libs.versions.toml)에서 확인할 수 있습니다.

### New 🎉

`koin-core`
- `KoinContext`는 이제 다음을 포함합니다:
    - `fun loadKoinModules(module: Module, createEagerInstances: Boolean = false)`
    - `fun loadKoinModules(modules: List<Module>, createEagerInstances: Boolean = false)`
- `koinApplication()` 함수는 이제 여러 형식을 사용합니다:
    - `koinApplication(createEagerInstances: Boolean = true, appDeclaration: KoinAppDeclaration? = null)`
    - `koinApplication(appDeclaration: KoinAppDeclaration?)`
    - `koinApplication(createEagerInstances: Boolean)`
- `KoinAppDeclaration` - 선언 스타일을 여는 데 도움이 됩니다.
- `KoinPlatformTimeTools` - JS를 위해 API Time을 사용합니다.
- iOS - `synchronized` API - Touchlab Lockable API를 사용합니다.

`koin-androidx-compose`
- 새로운 `KoinAndroidContext` - Android 환경에서 현재 Koin 컨텍스트에 바인딩합니다.

`koin-compose`
- 새로운 `KoinContext` - 현재 기본 컨텍스트로 컨텍스트를 시작합니다.

`koin-ktor`
- 이제 Ktor 인스턴스를 위해 격리된 컨텍스트를 사용합니다 (기본 컨텍스트 대신 `Application.getKoin()` 사용).
- Koin 플러그인은 새로운 모니터링 기능을 도입합니다.
- `RequestScope` - Ktor 요청에 스코프 인스턴스를 허용합니다.

### Experimental 🚧

`koin-android`
- `ViewModelScope` - ViewModel 스코프를 위한 실험적 API를 도입합니다.

`koin-core-coroutines` - 백그라운드에서 모듈을 로드하는 새로운 API 도입

### Deprecation ⚠️

`koin-android`
- `getLazyViewModelForClass()` API는 매우 복잡하며, 기본 전역 컨텍스트를 호출합니다. Android/Fragment API를 고수하는 것을 선호합니다.
- `resolveViewModelCompat()`은 `resolveViewModel()`을 선호하여 더 이상 사용되지 않습니다.

`koin-compose`
- 함수 `get()`과 `inject()`는 `koinInject()`를 선호하여 더 이상 사용되지 않습니다.
- 함수 `getViewModel()`은 `koinViewModel()`을 선호하여 더 이상 사용되지 않습니다.
- 함수 `rememberKoinInject()`는 `koinInject()`를 선호하여 더 이상 사용되지 않습니다.

### Breaking 💥

`koin-core`
- `Koin.loadModules(modules: List<Module>, allowOverride: Boolean = true, createEagerInstances : Boolean = false)`가 `Koin.loadModules(modules: List<Module>, allowOverride: Boolean = true)`를 대체합니다.
- 속성 `KoinExtension.koin`이 함수 `KoinExtension.onRegister()`로 이동되었습니다.
- iOS - `internal fun globalContextByMemoryModel(): KoinContext`를 사용하기 위한 `MutableGlobalContext`

`koin-compose`
- 함수 `KoinApplication(moduleList: () -> List<Module>, content: @Composable () -> Unit)`는 `KoinContext` 및 `KoinAndroidContext`를 선호하여 제거되었습니다.

## 3.4.3

:::note
Kotlin `1.8.21` 버전을 사용합니다.
:::

### New 🎉

`koin-core`
- Koin을 위한 확장 엔진 작성을 돕는 새로운 ExtensionManager API - `ExtensionManager` + `KoinExtension`
- `parameterArrayOf` 및 `parameterSetOf`로 매개변수 API 업데이트

`koin-test`
- `Verification` API - 모듈에서 `verify`를 실행하는 데 도움이 됩니다.

`koin-android`
- ViewModel 주입을 위한 내부 요소
- `AndroidScopeComponent.onCloseScope()` 함수 콜백 추가

`koin-android-test`
- `Verification` API - 모듈에서 `androidVerify()`를 실행하는 데 도움이 됩니다.

`koin-androidx-compose`
- 새로운 `get()`
- 새로운 `getViewModel()`
- 새로운 스코프 `KoinActivityScope`, `KoinFragmentScope`

`koin-androidx-compose-navigation` - 내비게이션을 위한 새로운 모듈
- 새로운 `koinNavViewModel()`

`koin-compose` - Compose를 위한 새로운 멀티플랫폼 API
- `koinInject`, `rememberKoinInject`
- `KoinApplication`

### Experimental 🚧

`koin-compose` - Compose를 위한 새로운 실험적 멀티플랫폼 API
- `rememberKoinModules`
- `KoinScope`, `rememberKoinScope`

### Deprecation ⚠️

`koin-compose`
- 지연(Lazy) 함수 사용을 피하면서 `inject()` 사용을 대체하는 함수 `get()`
- 지연(Lazy) 함수 사용을 피하면서 `viewModel()` 함수를 대체하는 함수 `getViewModel()`

### Breaking 💥

`koin-android`
- `LifecycleScopeDelegate`는 이제 제거되었습니다.

`koin-androidx-compose`
- `koinViewModel`을 선호하여 `getStateViewModel` 제거