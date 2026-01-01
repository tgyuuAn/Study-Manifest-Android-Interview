### 34. Canvas는 어떤 역할을 하나요?

- Canvas는 커스텀 그래픽, 애니메이션 및 시각 효과를 만들 수 있도록 개발자가 직접 원, 선, 도형 등을 그리거나 이미지를 렌더링 할 수 있는 일종의 스케치북과 같은 API임.
- 표준 UI 컴포넌트와 달리 Canvas는 DrawScope 인터페이스를 제공하여 드로잉 커맨드를 사용하여 렌더링을 세부적으로 제어할 수 있도록 함.
- Canvas는 Jetpack Cmopose의 드로잉 시스템 매커니즘에 기반하여 drawRect, drawCircle, drawPath, drawText, drawImage와 같은 함수를 통해 전통적인 XML의 Canvas와 유사한 형태로 동작함.

#### 사용 예제

```kotlin
@Composable
fun DrawCircleCanvas() {
    Canvas(modifier = Modifier.size(200.dp)) { // 고정 크기의 Canvas
        // Canvas 중앙에 파란색 원 그리기
        drawCircle(
            color = Color.Blue,
            radius = size.minDimension / 2, // Canvas 크기에 기반한 반지름
            center = center // Canvas 중앙 좌표
        )
    }
}
```

<br><br><br>

#### 기본 변환

- Canvas 컴포저블은 개발자가 동적이고 상호 작용 가능한 UI 컴포넌트를 만들 수 있도록 다양한 변환 및 드로잉 함수를 제공함.
- 크기 조절 (scale) : 지정된 배율을 드로잉 대상 확대 또는 축소
- 이동 (translate) : 드로잉 영역 내에서 X 및 Y 축을 따라 대상 이동
- 회전 (rotate) : 피벗 점 주위로 대상 회전
- 인셋 (inset) : 패딩을 적용하여 드로잉 경계 조정.
- 다중 변환 (withTransforma) : 더 나은 성능을 위해 단일 작업에서 여러 변환 결합.
- 텍스트 그리기 (drawText) : 정밀한 위치 지정 및 커스텀로 텍스트 수동 렌더링.

<br><br><br>

### 35. graphicsLayer를 어떻게 활용하나요?

- graphicsLayer는 개발자가 컴포저블에 변환, 클리핑 및 합성 효과를 적용할 수 있도록 하는 유용한 Modifier.
- 컴포저블을 별도의 드로잉 레이어로 렌더링하여 작동하며, 격리된 렌더링, 캐싱 및 오프스크린 래스터화와 같은 최적화를 가능하게 함.
- 수동적으로 드로잉을 제어하는 Canvas와는 달리 graphicsLayer는 컴포저블의 구성 가능성을 유지하여 훨씬 선언적인 방법으로 컴포넌트의 모양을 수정할 수 있음.

 #### graphicsLayer 작동 방식

 - 컴포저블이 Modifier.graphicsLayer로 래핑되면 모든 드로잉 작업이 나머지 UI와는 별도로 분리되는 격리된 레이어를 생성함.
 - 즉, 크기 조절, 이동, 회전, 알파 변경 및 클리핑과 같은 변환을 주변 컴포저블에 영향을 주지 않고 적용할 수 있음.
 - 또한 graphicsLayer는 하드웨어 가속을 사용하므로 과도한 recomposition을 트리거 하지 않고도 이러한 효과를 효율적으로 적용할 수 있음.

<br><br><br>

#### 합성 전략 (Compositiong Strategies)

- graphicsLayer는 콘텐츠가 렌더링되는 방식을 결정하는 다양한 합성 전략을 제공함.
1. Auto (기본값) : 속성에 따라 렌더링을 자동으로 최적화함.
2. Offscreen : 합성하기 전에 콘텐츠를 오프스크린 텍스처로 렌더링함.
3. ModulateAlpha : 전체 레이어 대신 드로잉 작업별로 alpha를 적용함.

```kotlin
@Composable
fun OffscreenBlendEffect() {
    Image(
        painter = painterResource(id = R.drawable.skydoves_image),
        contentDescriptioin = "Blended Image",
        modifier = Modifier
            .size(200.dp)
            .graphicsLayer {
                compositingStrategy = CompositingStrategy.Offscreen // 오프스크린 전략
                // 여기에 BlendMode 등 적용 가능
            }
    )
}
```
- BlendMode와 같은 효과가 다른 컴포넌트에 영향을 주지 않고, 해당 컴포저블에만 적용되도록 보장함.

<br><br><br>

#### 컴포저블로부터 Bitmap 휙득하기

- Compose 1.7.0부터 graphicsLayer를 사용하여 컴포저블을 비트맵으로 추출할 수 있음.
- 안드로이드의 전통적인 Bitmap은 구조가 준수하게 잡혀있는 편이기 때문에, 여전히 Composable 함수로부터 Bitmap을 휙득하여 여러 형태로 사용해볼 수 있음.
- 가령, 가우시안 블러 효과를 구현하는 등 여전히 이미지에 산술적인 연산이 필요한 경우에 유의미할 수 있음.
- 전통적인 XML에는 View 객체로부터 Bitmap을 휙득할 수 있는 방법이 존재했던 반면에, Jetpack Compose는 최근 들어서야 graphicsLayer API를 통해 구현할 수 있게 되었음.

```kotlin
val coroutineScope = rememberCoroutinerScope()
// rememberGraphicsLayer()를 사용하여 GraphicsLayerController 생성 및 기억
val graphicsLayer = rememberGraphicsLayer()

Box(
    modifier = Modifier
        // drawWithContent를 사용하여 그리기 제어
        .drawWithContent {
            graphicsLayer.record {
                // 원래 콘텐츠 그리기
                this@drawWithContent.drawContent()
            }

            // 기록된 레이어 그리기
            drawLayer(graphicsLayer)
        }
        .clickable {
            coroutineScope.launch {
                val bitmap = try {
                    // 기록된 GraphicsLayer에서 ImageBitmap 생성 시도
                    graphicsLayer.toImageBitmap()
                } catch (e: Exception) {
                    // 오류 처리 (예: 캡쳐 실패)
                    Log.e("GraphicsLyaer", "Faiiled to capture bitmap", e)
                    null
                }

                // 비트맵 저장 또는 공유
                bitmap?.let { /* 비트맵 처리 로직 */ }
            }
        }
        .background(Color.White)
) {
    Text("Hello Compose", fontSize = 26.sp)
}
```

<br><br><br>

### 36. Jetpack Compose에서 애니메이션을 어떻게 구현하나요?

- Jetpack Compose는 UI 상태 간의 부드럽고 시각적인 효과와 함께 전환을 가능하게 하는 선언적 애니메이션 시스템을 제공함.
- Compose에서 기본적으로 제공하는 애니메이션 API는 개발자가 컴포저블 가시성, 콘텐츠 변경, 크기 조정 및 속성 전환 등에 쉽게 애니메이션을 구현할 수 있도록 함.
- 가장 주목할 만한 부분은 전통적인 xml에서의 애니메이션보다 월등히 구현하기 쉽고, 잘 활용한다면 최적의 성능을 유지하면서 사용자 경험을 향상시킬 수 있음.

<br><br><br>

#### AnimatedVisibility로 가시성 애니메이션 구현하기

- AnimatedVisibility를 사용하면 컴포저블이 나타나고 사라지는 페이드 인/아웃 전환 애니메이션을 구현할 수 있음
- 컨텐츠가 나타날 때는 페이드 인에 추가적인 효과를 적용할 수 있고 사라질 때는 페이드 아웃에 축소 효과 등을 적용할 수 있음.

<br><br><br>

### 37. 화면 간 내비게이션을 어떻게 구현하나요?

- 최근 best practive라고 일컫는 SAA가 선호됨에 따라 xml 기반 프로젝트에서도 네비게이션 시스템은 안드로이드 개발에 있어 필수적인 부분으로 자리하고 있음.
- Jetpack Compose에서는 Fragment 개념이 존재하지 않기 때문에, 네비게이션 스택, 컨트롤러 및 UI 상태를 관리하기 위한 전용 네비게이션 시스템이 필요함.
- Compose 프로젝트에서 네비게이션을 구현하는 두 가지 기본 접근 방식은 수동적으로 네비게이션 시스템을 구현하는 방법과 스택 처리 및 상태 보존을 내부적으로 모두 처리해주는 Jetpack Compose Navigation 라이브러리를 사용하는 방법이 있음.

<br><br><br>

#### 네비게이션 직접 구현하기

- 컴포저블의 상태를 관리하고 보존하기 위해 고유 키를 기반으로 rememberSaveable과 함께 작동하는 Compose Runtime API인 SaveableStateHolder를 사용하여 수동으로 네비게이션 시스템을 구현할 수 있음,.
- 컴포저블이 컴포지션에 제거될 때(가령, 새 화면으로 이동할 때) 해당 상태는 자동으로 저장되고 컴포지션에 다시 진입할 때 복원됨.

```kotlin
@Composable
fun <T : Any> Navigation(
    currentScreen: T,
    modifier: Modifier = Modifier,
    content: @Composable (T) -> Unit
) {
    // SaveableStateHolder 생성 및 기억
    val saveableStateHolder = remembeSaveableStateHolder()

    Box(modifier) {
        // AnimatedContent를 사용하여 화면 전환 시 애니메이션 효과 적용
        AnimatedContent(targetState = currentScreen, label = "NavigationContent") {
            targetScreen ->
            // SaveableStateProvider로 현재 화면 콘텐츠 래핑 (key로 화면 상태 구분)
            saveableStateHolder.SaveableStateProvider(key = targetScreen) {
                content(targetScreen) // 실제 화면 콘텐츠 렌더링
            }
        }
    }
}

// 메인 컴포저블 (화면 전환 버튼 및 Navigation 호출)
@Composable
fun SaveableStateHolderExample() {
    var screen by rememberSaveable { mutableStateOf("screen1") }

    Column {
        // 화면 전환 버튼
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceEvenly,
        ) {
            Button(onClick = { screen = "screen1" }) { Text("Go to screen1") }
            Button(onClick = { screen = "screen2" }) { Text("Go to screen2") }
        }

        // Navigation 컴포저블을 사용하여 화면 표시
        Navigation(currentScreen = screen, modifier = Modifier.fillMaxSizxe()) {
            currentScreen ->
            when (currentScreen) {
                "screen1" -> Screen1()
                "screen2" -> Screen2()
            }
        }
    }
}
```

<br><br><br>

#### Jetpack Compose Navigation

- Jetpack Compose는 Navigation 시스템의 기존 인프라를 활용하며너 컴포저블 간의 원활한 네비게이션을 가능하게 하는 Compose용 네비게이션 라이브러리를 제공함.
- 구조화된 네비게이션 관리, 상태 처리 및 딥 링킹 기능, 안전한 인수 전달 기능 들을 제공하여 Compose에서 네비게이션 시스템이 필요한 경우 해당 라이브러리를 통해 대부분의 문제는 해결할 수 있음.
- NavHost : 네비게이션 그래프를 정의하고 컴포저블 화면을 라우트와 연결함
- NavController : 네비게이션 스택을 관리하고 목적지 간의 네비게이션을 처리함
- Composable Destinations : 네비게이션 그래프 내의 개별 화면임.

<br><br><br>

### 40. 스크린샷 테스트란 무엇이며, UI 일관성을 보장하는 데 어떻게 도움이 되나요?

- 스크린샷 테스트는 실제 기기에서 앱을 실행하지 않고 UI 렌더링 결과를 확인하는 효과적인 방법.
- 특히 코드 리뷰 단계에서 컴포넌트 변경사항에 대한 새로운 스크린샷을 이전 스크린샷과 비교하여 변경 사항을 시각적인 형태로 감지할 수 있어 수정 사항을 쉽고 빠르게 식별할 수 있음.
- Jetpack Compose에서 스크린샷 테스트를 수행하는 세 가지 방법이 있음 Google의 공식 Gradle 플러그인과 오픈소스 커뮤니티에서 유명한 Paparazzi 및 Roborazzi임.

<br><br><br>

#### Compose 스크린샷 테스트 플러그인

- Google에서 공식적으로 제공하는 Compose 스크린샷 테스트 플러그인은 Compose Preview와 함께 작동하여 개발자가 UI 스냅샷을 원활하게 생성하고 비교할 수도 있도록 함.
- 이 방법은 UI 일관성을 확인하고 의도하지 않은 레이아웃 변경 사항을 감지하는 데 유용함.
- 스크린샷 테스트는 UI 스냅샷을 캡처하고 이전에 생성되었던 스냅샷 이미지와 비교함.
- Compose Preview 스크린샷 테스트 도구를 사용하여 다음 항목들을 수행할 수 있음
- 스크린샷 테스트를 위한 Compose 미리 보기 선택
- 비교를 위한 참조 이미지 생성
- UI 변경 사항 자동 감지 및 HTML 리포트 생성
- 테스트 범위를 확장하기 위해 uiModel 및 fontScale과 같은 @preview 매개변수 사용
- 모듈화를 위해 screenshotTest 소스 세트를 사용하여 테스트 구성

<br><br><br>

#### Paparazzi

- Paparazzi는 Cash App에서 개발한 오픈 소스 라이브러리로, 에뮬레이터나 실제 기기 없이 스크린샷 테스트를 가능하게 함.
- 모든 작업은 JVM에서 실행되므로 UI 스냅샷을 캡처하는 빠르고 효율적인 방법임.
- Paparazzi는 JVM에서 직접 Compose UI를 렌더링하고 비교를 위해 픽셀 단위로 완벽한 스크린샷을 캡처하는 방식으로 작동함.

```kotlin
class LaunchViewTest {
    @get:Rule
    val paparazzi = Paparazzi(
    deviceConfig = PIXEL_5, // 기기 종류 설정
    theme = "android:Theme.Material.Light.NoActionBar" // 테마 설정
    // ... 더 많은 옵션은 문서 참조
    )

    @Test
    fun launchView() {
        // XML 레이아웃 인플레이트
        val view = paparazzi.inflate<LaunchView>(R.layout.launch)
        // 또는 프로그래밍 방식으로 View 생성
        // val view = LaunchView(paparazzi.context)

        view.setModel(LaunchModel(title = "paparazzi")) // 모델 설정
        paparazzi.snapshit(view) // 스냅샷 생성
    }

    @Test
    fun launchComposable() {
        // Composable 스냅샷 생성
        paparazzi.snapshot {
            MyComposable()
        }
    }
}
```

<br><br><br>

#### Roborazzi

- Roborazzi는 Jetpack Compose를 포함한 안드로이드 스크린샷 테스트를 위해 설계된 오픈 소스 라이브러리임.
- 스냅샷 비교를 통해 UI 상태를 캡처하고, UI 변경 사항을 확인하기 위한 간단하고 유연한 API를 제공함.
- Roborazzi는 Roboletric 위에서 동작하여, Hilt와 함께 테스트를 실행할 수 있고 보다 현실적인 환경에서 UI 컴포넌트와 상호 작용할 수 있음.
- Robolectric는 JVM 환경이 아니라 에뮬레이터나 실제 디바이스에서 실행되는 환경이기 때문에, 실제 렌더링 환경에서의 스크린샷을 캡처함으로써 paparazzi가 커버하지 못하는 한계점을 개선하여 런타임 의존성 주입 및 기타 시스템 수준의 상호 작용 등 기존 호환성을 보장하면서 테스트 프로세스를 더 효율적이고 신뢰할 수 있게 만듬

<br><br><br>

### 41. Jetpack Compose에서 접근성을 어떻게 보장하나요?

#### 접근성 테스트하기

- Accessibility Sacnner 또는 Compose UI 테스트의 AccessibilityTestRule을 사용하여 접근성 레이블, 역할 및 계층 구조를 검증할 수 있음.
- 또한 Compose는 적절한 접근성 동작을 보장하기 위해 테스트에서 시멘틱스 단언을 허용함.

```kotlin
// contentDescription이 "전송"인 노드가 존재하는지 검증
composeTestRule.onNodeWithContentDescription("Send").assertExists()
```

