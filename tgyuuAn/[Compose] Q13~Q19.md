### Q13.) rememberdhk rememberSaveable의 차이점은 무엇인가요?

- Jetpack Compose에서 상태 관리는 UI가 데이터 변경에 따라 동적으로 업데이트되도록 하는 핵심적인 개념임.
- remember와 rememberSaveable은 모두 recomposition으로부터 상태를 유지하도록 하는 API이지만 서로 다른 목적을 가지고 있으며 각자 다른 시나리오에 적합함.

<br><br><br>

#### remember 이해하기

- 목적 : remember API는 메모리에 값을 저장하고 recomposition가 발생했을 경우 메모리에 저장된 값을 꺼내와 상태를 유지함. 그러나 화면 회전이나 프로세스 재시작과 같은 구성 변경중에는 상태를 유지하지 않음.
- 유즈 케이스 : 상태가 구성 변경 후에도 유지될 필요가 없을 때 remember를 사용함. 예를 들어 화면이 회전되거나 사용자가 언어 설정 등을 바꾸었을 경우, 정보가 날아가도 상관없는 상태의 경우 remember가 적합함.
- 동작 : remember는 현재 컴포지션 생명주기 내에서만 상태를 저장하므로 기기가 회전화면 count 변수가 0으로 재설정됨.

<br><br><br>

#### rememberSaveable 이해하기

- 목적 : rememberSaveable API는 구성 변경 시에도 상태를 유지하여 remember 보다 더 넓은 범위에서 상태를 저장하고 복원함. 안드로이드 SDK의 Bundle에 저장할 수 있는 값에 한하여 자동으로 저장하고 복원함
- 유스 케이스 : 유저 인풋 입력이나 내비게이션 상태와 같이 구성 변경 후에도 유지되어야하는 상태에 대해서는 rememberSaveable을 사용해야 함
- 동작 : text 같은 화면 회전이나 구성 변경 시에도 유지되어 원활한 사용자 경험을 보장함

<br><br><br>

#### 사용 시기

- 애니메이션이나 임시적은 UI 상태와 같이 현재 컴포지션을 넘어서 유지될 필요가 없는 일시적인 상태는 remember를 사용함
- 사용자 입력, 선택 상태 또는 양식 데이터와 같이 구성 변경 시에도 유지되어 더 나은 사용자 경험을 제공할 수 있는 상태에는 rememberSaveable을 사용함.

<br><br><br>

#### remember 및 rememberSaveable 내부 구조

- remember 내부 구조

```kotlin
@Composable
inline fun <T> remember(corssinline calculation: @DisallowComposableCalls () -> T): T = currentComposer.cache(false, calculation)
```

- 코드에서 볼 수 있듯이 remember는 내부적으로 composer 인스턴스에서 cache 함수를 호출함.
- cache 함수가 구현되는 방식은 아래와 같음.

```kotlin
@ComposeCompilerApi
inline fun <T> Composer.cache(invalid: Boolean, block: @DisallowComposableCalls () -> T): T {
    @Suppress("UNCHECKED_CAST"
    return rememberdValue().let {
        if (invalid :: it === Composer.Empty) { // 값이 유효하지 않거나 초기화되지 않았는지 확인
            val value = block() // 계산(remember 매개변수인 `calculation`) 블록 실행
            updatedRememberedVaklkue(value) // 컴포지션 데이터에 값 저장
            value
        } else it // 이전에 기억된 값 반환
    } as T
}
```

- 위의 코드는 remember가 내부적으로 어떻게 작동하는지 보여줌
- Compose 컴파일러 플러그인 API와 상호 작용하여 컴포지션 데이터에 값을 캐시함.
- 구체적으로, 값이 유효하지 않거나 초기화되지 않았는지 확인함.
- 초기화되지 않았다면, 제공된 블록 람다 함수를 사용하여 값을 계산하고 컴포지션 데이터에 저장한 다음 반환함.
- 그렇지 않으면 이전에 기억된 값을 단순히 복원하여 반환함.
- 이와 같은 메커니즘은 중복적인 계산을 피하면서 recomposition이 발생해도 값이 효율적으로 유지되도록 보장함

<br><br><br>

- rememberSaveable 내부 구조

```kotlin
@Composable
fun <T: Any> rememberSaveable(
    vararg inputs: Any?, // 상태 재계산을 트리거 하기 위한 입력 키
    saver: Saver<T, out Any> = autoSaver(), // 상태 저장 및 복원 로직
    key: String? = null, // 상태 저장을 위한 고유 키
    init: () -> T // 초기값 계산 람다
): T {
    // 현재 컴포지션 해시로부터 기본 키 생성
    val compositeKey = currentCompositeKeyhash
    // 사용자가 제공한 키 또는 생성된 키 사용
    val finalKey = if (!key.isNullOrEmpty()) {
        key
    } else {
        compositeKey.toString(MaxSupportedRadix)
    }

    @Suppress("UNCHECKD_CAST")
    (saver as Saver<T, Any>) // Saver 타입 캐스팅

    // 현재 SaveableStateRegistry 가져오기
    val registry = LocalSaveableStateRegistry.current

    // 상태를 관리할 Holder 생성 및 기억
    val holder = remember(saver, registry, finalKey) { // 키 변경 시 재실행
        // 레지스트리 복원 시도 또는 init으로 초기화
        val restored = registry?.consumedRestored(finalKey)?.let { saver.restore(it) }

        val finalValue = restored ?: init()
        SaveableHolder(saver, registry, finalKey, finalValue, inputs)
    }

    // 입력 키가 변경되지 않았다면 저장된 값 사용, 아니면 init 재실행 
    val value = holder.getValueIfInputsDidntChange(iputs) ?: init()

    // recomposition 시 상태 업데이트 및 레지스트리에 등록
    SideEffect {
        holder.update(saver, registry, finalKey, value, inputs)
    }

    return value
}
```

1. 키 생성 -> 2. 상태 복원 -> 3. 기본값 초기화 -> 4. Saveable Holder -> 5. 입력 변경 처리 -> 6. 사이드 이펙트

### Q14.) 컴포저블 함수 내에서 안전하게 코루틴 스코프를 생성하는 방법은 무엇인가요?

- Jetpack Compose에서 RememberCoroutineScope는 컴포저블 함수 내에서 코루틴 스코프를 안전하게 생성하고 관리하는 공식적으로 권장하는 접근 방식.
- 이는 코루틴 스코프가 컴포지션에 연결되도록 보장하여 잠재적인 메모리 누수 및 부적절한 리소스 사용을 방지함.

<br><br><br>

#### rememberCoroutineScope를 사용하는 이유

- Jetpack Compose의 rememberCoroutineScope는 컴포저블이 컴포지션을 벗어날 때 활성 중인 코루틴 스코프를 자동으로 취소함.
- 이는 내부적으로 컴포저블 함수의 내부 생명주기를 인지하게 작동하고 있음을 의미하고, 이를 통해 개발자는 수동적으로 생명주기를 관리할 필요 없이 컴포저블에서 코루틴을 안전하게 런칭 및 취소할 수 있음.

<br><br><br>

#### rememberCoroutineScope 내부 구조

```kotlin
@Composable
inline fun rememberCoroutineScope(
    corssinline getContext: @DisallowComposableCalls () -> CoroutineContext = { EmptyCoroutineContext }
): CoroutineScope {
    val composer = currentComposer
    // CompositionScopedCoroutineScopeCanceller 인스턴스를 remember로 감싸서 유지
    val wrapper = remember {
        CompositionScopedCoroutineScopeCanceller(
            createCompositionCoroutinrScope(getContext(), composer)
        )
    }

    return wrapper.coroutineScope
}
```

<br><br><br>

#### 1. CompositionScopedCoroutineScopeCanceller

```kotlin
internal class CompositionScopedCoroutineScopeCanceller(
    val coroutineScope: CoroutineScope
): RememberObserver { // 컴포지션 생명주기 관찰
    override fun onRememberd() {
        // nothing
    }

    overrie fun onForgotten() {
        // 컴포지션을 벗어날 때 현재 실행 중인 스코프 취소
        coroutineScope.cancel(LeftCompositionCancellationException())
    }

    overrie fun onAbandoned() {
        // 컴포지션이 버려질 때 현재 실행 중인 스코프 취소
        coroutineScope.cancel(LeftCompositionCancellationException())
    }
}
```

<br><br><br>

#### 2. createCompositionCoroutineScope

```kotlin
internal fun crerateCompositionCoroutineScope(
    coroutineContext: CoroutineContext,
    composer: Composer,
) = if (coroutineContext[Job] != null) {
    // Job이 이미 존재하면 예외 발생 (독립 스코프 보장)
    CoroutineScope(
        Job().apply {
            completeExceptionally(
                IllegalArgumnetException("CoroutineContext supplied to rememberCoroutineScope may not include a parent job")
            )
        }
    )
} else {
    // Composer의 applyCoroutoineContext와 새 Job 결합하여 스코프 생성
    val applyContext = composer.applyCoroutineContext
    CoroutineScope(applyContext + Job(applyContext[Job]) + coroutineContext)
}
```

- 상위 Job 제한 : 제공된 CoroutineContext에서 Job을 포함하고 있는 경우 예외가 발생함. 이는 rememberCoroutineScope가 기존의 상위 Job에 종속되는 것잉 아닌 독립적인 스코프를 생성하도록 보장함
- Job 관리 : 스코프를 관리하기 위해 새로운 Job이 코루틴 컨텍스트에 추가됨
- Composer와의 통합 : 함수는 composer.applyCoroutineContext를 새로 생성된 Job과 결합하여 코루틴 스코프를 설정함

#### rememberCoroutineScope 내부 작동 방식

1. 스코프 생성 : createCompositionCouroutineScope를 사용하여 새 코루틴 스코프가 생성되고, CompositionScopedCoroutineScopeCanceller에 전달됨.
2. 컴포지션 생명주기 인지 : CompositionScopedCoroutineScopeCanceller는 컴포지션 생명주기를 관찰함. 컴포저블이 컴포지션에서 제거되면 메모리 누수를 방지하기 위해 스코프가 취소됨.

<br><br><br>

### Q15.) 컴포저블 함수 내에서 발생하는 사이드 이펙트를 어떻게 처리하나요?

#### 1. LaunchedEffect : 컴포저블 범위 내에서 suspend 함수 실행하기

- LaunchedEffect는 컴포저블의 컴포지션 생명주기 내에서 새로운 코루틴 스코프를 생성하고 실행함.
- 생성된 코루틴은 LaunchedEffect에 전달된 키가 변경되면 취소되고 다시 시작됨
- 컴포저블이 컴포지션에 진입할 때 수행해야 하는 데이터 로딩하기, 애니메이션 시작 또는 이벤트 수신과 같은 작업에 유용함.

주요 특징은 아래와 같음.
- 컴포저블이 컴포지션에 진입할 때 한 번 실행됨
- 키가 변경되면 자동으로 취소되고 다시 시작됨
- 컴포지션 생명주기를 인지함. 즉, 컴포저블이 컴포지션을 떠날 때 코루틴 작업이 자동으로 취소됨.

- 가령, 리스트에서 특정 아이템을 클릭할 때 네트워크에서 아주 가벼운 데이터를 추가적으로 가져와야 하는 경우 및 애니메이션을 실행해야하는 경우 LaunchedEffect를 사용하여 처리할 수 있음. 또한, LaucnhedEffect에 매개변수로 전달된 키 값이 변경될 때마다 작업을 취소하고 다시 실행할 수 있음.

```kotlin
val selectedPoster: Poster? by remember { mutableStateOf(null) }

// selectedPoster가 변경될 때마다 LaunchedEffect 재실행
LaunchedEffect(key1 = selectedPoster) {
    selectedPoster?.let { poster ->
        // 네트워크에서 추가 정보 가져오기 위해 ViewModel에 이벤트 전송
        viewModel.fetchPosterDetails(poster.id)

        // Fade 애니메이션 실행
        .. // suspend 함수 실행
    }
}
```

<br><br><br>

#### 2. DisposableEffect : 메모리 해지 등이 필요한 사이드 이펙트 실행에 적합

- DisposableEffect는 컴포저블 라이프 사이클에 따라 리소스 해지 및 실행 중인 태스크를 정리하기 위해 사용됨.
- LaunchedEffect와 달리 컴포저블이 컴포지션을 떠날 때 리소스를 해제하기 위해 DisposableEffectScope를 제공하고 해당 스코프 안에서 onDispose 람다를 통해 해지 작업을 수행할 수 있음.

<br><br><br>

#### 3. SideEffect : Compose의 상태를 non-Compose 코드로 실행하기

- SideEffect는 매 recomposition 직후에 적용해야 하는 작업을 실행하는 데 사용됨.
- 컴포저블이 recompose된 후 실행을 보장하므로 ViewModel이나 외부 라이브러리의 UI 상태 업데이트와 같이 컴포지션의 일부가 아닌 외부 시스템과 Compose 상태를 동기화하는 데 적합함. 

주요 특징은 아래와 같음.
- 매 recomposition 후 실행이 보장됨
- non-Compose 컴포넌트와의 상태 동기화에 유용함

<br><br><br>

### Q16.) rememberUpdateState는 왜 사용하고 어떻게 작동하나요?

- rememberUpdateState 함수는 컴포지션 컨텍스트 내에서 람다 함수에 대한 상태 업데이트를 안전하게 처리하는 런타임 API임.
- 최소 컴포지션을 통해서 생성되었더라도 람다나 콜백의 상태 값에 대해 항상 최신 상태를 유지하도록 보장함.
- 컴포저블에서 콜백이나 람다를 생성할 때 해당 콜백 내에서 참조된 상태 값은 함수가 이미 생성된 경우 자동으로 업데이트되지 않을 수 있음. 이를 위해 rememberUpdateState가 유용하게 사용됨.

```kotlin
@Composable
fun LandingScreen(onTimeout: () -> Unit) {
    val currentOnTimeout by rememberUpdatedState(onTimeout)

    LaunchedEffect(true) {
        delay(SplashWaitTimeMillis)
        currentOnTimeout()
    }

    /* Landing screen content */
}
```

<br><br><br>

### Q17.) produceState의 목적은 무엇이며 어떻게 작동하나요?

- produceState 함수는 새로 시작된 코루틴에 의해 값이 생성되는 State 객체를 만드는 데 도움이 됨.
- 비동기적으로 불러와야 하는 데이터나 코루틴을 활용하여 연산이 필요한 동작과 Compose 간의 다리역할을 함.
- 이는 비동기 작업에 의존하는 상태를 관리하거나 non-Compose 상태를 Compose 상태로 변환해야 할 때 특히 유용함.
- 컴포저블이 관찰할 수 있는 State 객체를 생성하고, 상태 값을 업데이트하기 위해 생상자 코루틴을 실행하며, 컴포저블이 컴포지션을 떠날 때 코루틴 스코프를 자동으로 취소함.

<br><br><br>

### Q18.) snapshotFlow를 사용해 본 경험이 있을까요? 사용 시 주의 사항은 무엇인가요?

- snapshotFlow는 Compose의 상태를 Flow로 변환하는 함수임
- Compose가 내부적으로 상태 변경을 효율적으로 관리하고 관찰하는 데 사용하는 snapshot 시스템 내에서 변경 사항을 관찰함.
- 관찰된 상태가 변경될 때마다 Flow는 업데이트된 값을 내보냄.

<br><br><br>

#### snapshotFlow의 주요 특징

- 상태 관찰 : Snapshot 시스템을 사용하여 Compose의 상태 변화를 관찰함
- 스레드 안정성 : 상태 읽기 및 쓰기가 Compose의 스냅샷 스코프 내에서 발생하도록 보장하여 경쟁 조건을 방지함
- 유휴 건너뛰기 : 상태 값이 변경될 때만 Flow 값으로의 방출이 발생하고 recomposition이 발생하지 않는 중에는 업데이트를 건너뛰도록 보장함.
- 취소 인지 : Flow를 수집하는 코루틴이 취소될 때 구독 또한 자동으로 취소하여, 컴포지션의 생명주기를 인지하는 형태로 동작하여 부적절한 메모리 누수를 방지함.

<br><br><br>

#### 유스 케이스

- 코루틴과의 인터페이스 : 변환, 플로우 결합 또는 필터링과 같은 추가적인 Flow의 연산 작업이 필요한 경우 Compose의 상태를 코루틴 플로우로 변경하여 여러 가지 이점을 얻을 수 있음.
- 비 UI 사이드 이펙트 : 애널리틱스 이벤트 전송이나 백엔드 호출이 필요한 경우와 같이 UI에 직접적으로 관련이 없는 사이드 이펙트 작업을 수행하기에 적합함.

<br><br><br>

#### 명심해야 할 사항

- 스레드 안정성 및 수집 : 수집된 플로우가 일반적으로 LaunchedEffect를 사용하는 코루틴 스코프 내에서 처리되도록 하여 컴포지션 외부에서 수집하는 것을 방지하고 메모리 누수를 방지함
- 방출 빈도 : snapshotFlow는 람다 내에서 읽은 스냅샷 상태가 변경될 때마다 값을 방출하며, 이는 생각보다 자주 발생할 수 있음. distinctUntiillChanged() 또는 debounce()와 같은 연산자를 사용하여 불필요한 방출을 줄이고 성능을 최적화함.
- 스냅샷 격리: Flow는 Compose의 스냅샷 시스템에 최대한 동기화되어 동작하는데, 즉, 값은 격리된 스냅샷을 기반으로 방출됨. 스냅샷 생명주기 내에서 예상되는 동작을 보장하기 위해 다른 Flow 또는 suspend 함수와 결합할 때는 주의해야 함.

<br><br><br>

### Q19.) derivedStateof가 필요한 시나리오는 무엇이고, recomposition 최적화에 어떻게 도움이 되나요?

- derivedStateOf는 하나 이상의 상태 객체에서 파생된 값을 계산하는 컴포저블 API임.
- 종속 상태 중 하나가 변경될 떄만 파생된 값이 다시 계산되도록 보장하여 반응형 상태 관계를 관리하는 효과적인 도구임.
- 주요 특징 중 하나는 종속 상태가 자주 업데이트되더라도 계산된 값 자체가 변경될 때만 recomposition을 트리거하여 recomposition을 최적화하는 것임.
- 이로 인해 derivedStateOf는 빈번한 상태 업데이트가 있는 시나리오에서 성능을 개선하고 불필요한 recomposition을 방지하는 데 유용함.
- derivedStateOf는 중복된 recomposition을 방지하도록 설계되었지만 연산 작업이 요구되기 때문에 약간의 오버헤드를 동반함. 따라서 recomposition 방지가 중요한 상황에서만 신중하게 사용하여 파생된 상태 유지 관리의 추가 비용보다 이점이 크도록 해야함.

<br><br><br?

#### 명심해야 할 주요 사항

- 파생된 상태가 recomposition으로부터 값을 유지하도록 하려면 항상 remember와 함께 derivedStateOf를 사용함
- 원활한 성능을 보장하기 위해 derivedStateOf에 아주 무거운 연산식을 사용하지 않는 것이 좋음. 오히려 해당 계산을 통한 오버헤드가 recomposition을 통해 UI를 업데이트하는 비용보다 더 크면 성능에 역효과가 날 수 있음.
- 고로, 불필요한 recomposition을 피하는 것이 중요한 경우에만 신중하게 사용하는 것이 좋음.
