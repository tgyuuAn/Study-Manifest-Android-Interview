### Q6.) 안전성 개선을 통해 Compose 성능을 최적화한 경험이 있나요?

- Compose 성능 최적화는 컴포저블 함수를 안정화하고 불필요한 Recomposition을 최소화하는데 달려있음.
- 안정성은 Compose가 Recomposition 중에 어떤 함수를 건너뛸지 효과적으로 결정할 수 있도록 하여 UI 렌더링 효율성을 향상시킴.

<br><br><br>

#### Immutable Collections

- List 또는 Map과 같은 읽기 전용 컬렉션은 구현 과정에서 참조값이 아닌 내부의 아이템에 변경이 발생할 수 있기 때문에, Unstable 처리됨
- Compose 컴파일러는 아이템 항목들이 변경 불가능한지 가능한지에 대한 여부를 결정할 수 없으므로 recomposition의 정확성을 보장하기 위해 해당 인스턴스를 unstable로 처리함
- 안정성을 보장하기 위해 본질적으로 Stable한 kotlinx.collections.immutable 또는 Guava의 Immutable 컬렉션을 사용할 수 있음.
- ImmutableList 코드를 살펴보면 같은 List 인터페이스 인데, 왜 Stable이냐 하면, ImmutableList를 Compose 컴파일러 내부에서 해당 패키지를 stable로 처리하도록 하드코딩 되어있기 때문임.
- 이에 대해서 자세히 알아보고 싶으면 https://github.com/JetBrains/kotlin/blob/eadcce82e781d7be850118e82333893ab7cf10a9/plugins/compose/compiler-hosted/src/main/java/androidx/compose/compiler/plugins/kotlin/analysis/KnownStableConstructs.kt#L48 를 살펴보면 됨.

<br><br><br>

#### Lambda Stability

- 람다 표현식을 매개변수로 받은 컴포저블 함수는 Stable로 처리하지만, 외부 변수를 참조하는 지의 여부에 따라서 다르게 처리함

1. 값을 캡처하지 않는 람다 : 싱글톤으로 최적화되어 Stable로 간주되고, Recomposition을 트리거하지 않음
2. 값을 캡처하는 람다 : 외부 변수에 의존하는 람다는 동적으로 변경사항에 응답하기 위해 Remember를 사용하여 메모제이션됨.

### Q0.) JetpackCompose의 동작 구조는 어떻게 이루어져 있나요?

- Jetpack Compose는 선언적 접근 방식을 사용하여 네이티브 안드로이드 애플리케이션을 구축하기 위한 가장 최신의 UI 툴킷임.
- Compose는 Compose Compiler, Compose Runtime, Compose UI의 세 가지 주요 계층으로 이루어져 있음.
- 각 계층은 UI 코드를 상호 작용 가능한 애플리케이션으로 변환하는데 중요한 역할을 함.

<br><br><Br>

#### 안정성 구성 파일

- Comppose Compiler 1.5.5에 공개된 안정성 구성 파일을 사용하면 stable로 간주하고 싶은 클래스를 개발자가 임의로 지정할 수 있음

-  stable로 처리할 클래스의 패키지 명을 나열하는 compose_compiler_‐ config.conf 파일을 만듭니다.
- Compose 컴파일러가 해당 파일에 정의된 패키지 목록에 대해 recomposition을 건너뛸 수 있도록 build.gradle.kts에서 추가적 으로 설정이 필요합니다.
- 해당 기능은 특히 서드파티 클래스나 커스텀 타입을 stable 하게 만드는 데 유용합니다.
- 이를 활성화하면 래퍼 클래스를 수동으 로 만들 필요 없이 프로젝트 내에서 전역적으로 특정 클래스를 stable로 지정할 수 있습니다.

<br><br><br>

#### Strong Skipping Mode

- Compose Compiler 1.5.4에서 소개된 Strong Skipping Mode는 불안정한 매개변수를 포함하더라도, restartable로 분류된 컴포저블 함수에 대해 Recomposition 생략을 활성화함.
- Stable 매개변수가 있는 컴포저블 함수는 객체 동등성을 사용하여 값을 비교하는 반면, unstable 매개변수는 인스턴스 동등성을 사용하여 비교됨
- 일단 해당 기능이 활성화되면 모든 컴포저블 함수에 대해서 적용되기 때문에, 특정한 함수를 제외하려면 @NonSkippableomposable 어노테이션을 사용해야함

<br><br><br>

#### 갭 버퍼에서 링크 테이블로 마이그레이션

- AOSP 코드를 살펴보면, 안드로이드 팀은 갭 버퍼에서 링크 테이블 자료 구조로 마이그레이션 하고 있다.
- 링크 테이블은 연결된 노드를 사용하여 데이터를 구성하므로 요소의 효율적인 삽입, 삭제 및 재배열이 가능함.
- 해당 변경 사항은 SlotTable의 편집 성능을 향상시키면서, 동시에 현재 구현된 동작 방식의 효율성을 유지하는 것을 목표로 함.

<br><br><br>

### Q7.) 컴포지션이란 무엇이며 어떻게 생성하나요?

- 컴포지션은 컴포즈 컴파일러가 해석한 UI에 대한 정보를 실질적인 UI로 구체화하는 하나의 단계
- UI를 컴포저블의 트리 구조로 구성하며, Composer를 활용하여 트리를 동적으로 생성하고 관리
- 컴포지션은 상태를 기록하고 UI를 효율적으로 업데이트하기 위해 노드 트리에 필요한 변경 사항을 적용하며, 변경 사항을 적용하고 UI를 업데이트 하는 과정을 recomposition이라고함.

<br><br><br>

#### Composition 생성하기

- 컴포지션은 컴포저블 함수를 화면에 렌더링할 수 있게 UI 계층 구조로 변환하는 프로세스임.
- 이러한 과정은 Jetpack Compose가 작동하는 방식의 핵심으로, 런타임에서 상태의 변경 사항을 추적하고 UI를 효율적으로 업데이트할 수 있도록 함.
- 컴포지션을 생성하고 관리하는 방법은 다음과 같음

<br><br><br>

#### ComponentActivity.setContenet() 함수

- 컴포지션을 생성하는 가장 일반적인 방법은 ComponentActivity 또는 ComposeView에서 제공하는 setContent 함수를 사용하는 것.
- 이 함수는 컴포지션을 초기화하고 그 안에 표시될 콘텐츠를 정의함
- setContent는 컴포지션을 시작하는 역할을 하며, Compose UI의 진입점이 됨.
- setContent 내부적으로 ComposeView에 의존하며, CompseView.setContent를 호출하여 컴포지션을 초기화하고 생성함
- 함수는 먼저 Activity의 뷰 계층 구조에서 기존 ComposeView를 검색하고, 없으면 새 인스턴스를 생성
- 이 단계에서는 Activity에 ComposeView가 이미 있는 경우 불필요하게 새 인스턴스를 생성하는 대신 재사용할 수 있도록 보장함.
- 기존 ComposeView가 발견되면 새 부모 CompositionContext와 새 컴포저블 콘텐츠로 업데이트됨.

<br><br><br>

- ComposeUI는 본질적으로 전통적인 View 시스템, 특히 ComposeView 내에서 렌더링 됨.
- Jetpack Compose와 안드로이드 SDK에서 제공하는 렌더링 시스템 간의 다리 역할을 한다고 볼 수 있음.
- 따라서, Jetpack Compose는 기존의 전통적인 뷰 시스템과 "완전히 관련 없는 새로운 프레임워크다" 라고 생각할 수 없음.

<br><br><br>


### Q9.) Compose 성능 테스트를 항상 릴리스 모드에서 해야 하는 이유는 무엇인가요?

- Jetpack Compose 성능을 테스트할 때는 항상 R8 최적화가 활성화된 릴리스 모드에서 앱을 실행해야 함.
- 디버그 모드는 UI 코드에 대한 불필요한 해석, JIT 컴파일, LiveEditLiterals와 같은 개발자 도구와 관련된 것들이 추가 오버헤드를 유발
- Compose는 View 시스템보다 월등히 느린 경우가 많음.

<br><br><br>

#### 디버그 모드가 Compose에 미치는 영향

- Compose는 라이브러리("unbundled") 형태로 제공되므로, 디버그 앱을 실행할 때 런타임에 의하여 해석되고 컴파일됨.
- View 시스템의 경우는 안드로이드 OS와 함께 번들로 제공되어 미리 컴파일되는 반면에, Compose 라이브러리들은 디버그 모드에 Compose와 관련한 추가적인 해석 및 JIT 컴파일 오버헤드를 발생시킴.

<br><br><br>

#### Live Edit Literals 및 개발자 도구

- 디버그 빌드는 런타임 업데이트를 지원하기 위해 상수를 getter 함수로 대체하는 Live Edit Literals와 같은 개발자 기능을 활성화함.
- 이는 추가적인 산술 비용을 유발하고 최적화를 방해하여, 디버그 모드에서 recomposition 및 렌더링을 더 느리게 함.

<br><br><br>

#### Baseline Profiles가 중요한 이유

- 릴리스 모드에서 Jetpack Compose의 성능을 더욱 향상시키기 위해서는 Baseline Profiles을 함꼐 사용하는 것이 좋음.
- Baseline Profiles는 중요한 Compose 메서드를 사전에 컴파일하여 앱 시작 중 런타임 해석 및 JIT 컴파일을 피함.
- 보통 디버그 빌드의 경우 Baseline Profiles를 반영되지 않기 떄문에, 실제 릴리스 앱과의 실행 성능을 비교하면 체감될 정도로 월등히 개선되는 것을 느낄 수 있음.

<br><br><br>

### Q10.) Jetpack Compose에서 자주 사용하시는 Kotlin 관용구에 대해서 말씀해주세요.

#### 기본 매개변수

```kotlin
Text(
    text = "Hello, Android!",
    color = Color.Unspecified, // 기본값
    fontSize = TextUnit.Unspecified, // 기본값
)
```

<br><br><br>

### Q11.) 상태란 무엇이며 이를 관리하는 데 사용되는 API는 무엇인가요?

- Jetpack Compose는 State는 앱 시나리오에서 흔히 변경될 수 있는 값이자, UI에서 동적으로 반영되는 데이터를 나타냄.
- 흔히 사용되는 상태는 네트워크 오류에 대한 Snackbar 메시지 노출, 사용자 입력 또는 상호 작용으로 발생하는 애니메이션을 표시하기 위한 용도로 사용됨.

<br><br><br>

#### State와 Composition

- Jetpack Compose는 선언적 UI 접근 방식을 따르므로, UI 업데이트는 컴포저블이 변경된 매개변수를 통해 호출될 떄만 발생함.
- 이러한 동작은 composition 생명주기와 밀접하게 관련 있음.

<br>

- 초기 Composition : 컴포저블을 실행하여 UI트리가 처음 생성되고 렌더링되는 프로세스
- Recomposition : 상태 변경 시 트리거되며, recomposition은 관련 컴포저블을 업데이트하여 새로운 상태를 반영함

<br>

- Compose Runtime은 상태 변경 사항을 자동으로 추적하고, 안드로이드 View 시스템에서 UI를 호출하기 위해 사용하는 View.invalidate() 메서드와 유사한 동작을 개발자 대신하여 UI를 업데이트함.
- Recomposition은 업데이트된 상태를 반영해야 하는 컴포저블 함수에 대해서만 트리거됨.

<br><br><br>

### Q12.) 상태 호이스팅으로 어떤 이점을 얻을 수 있나요?

- 상태 호이스팅은 상태를 상위 수준의 컴포저블 함수로 끌어올리는 것을 의미
- 호이스팅이란, 기중기와 같은 것으로 무언가를 위쪽으로 끌어올리는 것을 의미.
- 상태 값과 상태 값을 업데이트하는 람다 함수를 컴포저블 매개변수로 전달하고 해당 값은 현재 컴포저블이 아닌 다른 호출자 쪽에서 관리하도록 하는 것

<br><br><br>

#### 상태 호이스팅의 장점

- 더 나은 재사용성 : 상태 호이스팅을 적용한 컴포저블은 Stateless 및 재상요 가능하게 만들 수 있음. 상태 및 이벤트 콜백을 전달함으로써 동일한 컴포저블을 특정 구현에 얽매이지 않고 다른 화면이나 컨텍스트에 사용될 수 있음.
- 단순화된 테스트 : 상태를 저장하지 않는 컴포저블은 매개변수로 전달된 상태 값에 전적으로 의존하므로 테스트하기 더 쉬움
- 더 나은 관심사 분리 : 상태 관리 로직을 부모 컴포저블 또는 ViewModel로 옮김으로써, UI 컴포넌트가 인터페이스 렌더링에만 집중하도록 함.
- 단방향 데이터 흐름 지원 : 단방향 데이터 흐름 아키텍처와 일치하여 상태가 단 하나의 공급원(SSOT) 에서 흐르도록 보장함. 여러 소스가 동일한 상태를 관리하려고 할 때 발생하는 예상치 못한 동작의 발생 가능성을 줄임
- 향상된 상태 관리 : 상태 호이스팅을 사용하면 ViewModel 또는 부모 컴포저블과 같은 상위 수준 컨테이너에서 상태를 중앙 집중화할 수 있음. 이를 통해 복잡한 UI 흐름을 관리하고 인스턴스 상태 저장 또는 상태 복원 관리와 같은 작업을 더 쉽게 처맇라 수 있음.
