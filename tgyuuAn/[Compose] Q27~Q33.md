### 27. Layout이란 무엇인가요?

- Layout 컴포저블은 자식 컴포저블의 크기 측정 및 위치 지정에 대한 완전한 제어를 제공하는 저수준 API임.
- 미리 정의된 동작을 제공하는 Column, Row 또는 Box와 같은 고수준 레이아웃 컴포넌트와 달리 Layout을 사용하면 개발자가 특정 요구 사항에 맞는 커스텀 레이아웃을 만들 수 있음.

#### Layout 작동 방식

- Layout 컴포저블은 자식 컴포저블의 크기 측정 및 위치 지정에 대한 완전한 제어를 제공하는 저수준 API임.
- 미리 정의된 동작을 제공하는 Column, Row 또는 Box와 같은 고수준 레이아웃 컴포넌트와 달리 Layout을 사용하면 개발자가 특정 요구 사항에 맞는 커스텀 레이아웃을 만들 수 있음.

<br><br><br>

#### Layout 작동 방식

- 측정 단계 (Measurement Phase) : 부모가 제공한 제약 조건에 따라 각 자식 컴포저블의 크기를 결정함.
- 배치 단계 (Placement Phase) : 사용 가능한 공간 내에 각 자식 컴포저블을 배치함

<br><br><br>

#### Layout의 주요 구성 요소

- 자식 측정 (Measuring Children) : measure() 함수는 부모로부터 제약 조건을 적용하여 각 자식 컴포저블을 측정하는 데 사용됨.
- 레이아웃 크기 결정 (Determining Layout Size) : layout() 함수는 레이아웃의 최종 너비와 높이를 정의함
- 자식 배치 (Placing Chihldren) : placeRelatinve() 함수는 레이아웃 내에서 각 자식 컴포저블이 배치될 위치를 결정함

<br><br><br>

#### Layout 유즈 케이스

- Layout 컴포저블은 Column, Row, Box와 같은 표준 레이아웃 컴포넌트가 달성하지 못하는 특수한 디자인 요구 사항에 필요한 커스텀 컴포넌트를 구현해야하는 시나리오에서 유용함.
- 자식 컴포저블이 측정되고 배치되는 방식에 대한 완전한 제어가 필요한 경우 Layout을 사용하여 커스텀 측정 및 배치 로직을 정의할 수 있음.
- 비표준 배열, 겹치는 요소 또는 콘텐츠에 따라 동적으로 크기가 조절되는 컴포넌트와 같이 복잡한 UI 구조를 구현할 때 특히 유용함.
- Layout은 제약 조건 및 측정에 직접 접근할 수 있으므로 개발자는 Recomposition 및 불필요한 재측정을 효율적으로 관리하여 UI 성능을 최적화할 수 있음.

<br><br><br>

#### SubcomposeLayout의 개념 및 동작 원리

- SubcomposeLayout은 레이아웃 내에서 동적 컴포지션을 허용하는 저수준 API임. UI 컴포넌트가 부모 레이아웃 패스와 독립적으로 하위 컴포넌트를 Recompose해야 할 때 주로 사용됨.
- 이는 자식 콘텐츠 크기가 비동기 데이터에 의존하거나 여러 측정 패스가 필요할 때 특히 유용함.

<br><br><br>

#### SubcomposeLayout 작동 방식

- SubcomposeLayout은 측정 단계 중 자식의 컴포지션을 제약 조건이 정해질 때까지 지연하는 것을 허용하여 특수한 경우에 유연성을 제공함.
- 일반적인 Layout 또는 LayoutModifier로 처리할 수 없는 컴포지션 중 부모가 가지는 제약 조건에 접근해야 하는 경우(가령, BoxWithConstraints와 같이 부모 컴포저블의 제약 조건을 자식 컴포저블이 알아야 하는 동작하는 형태)
- 한 자식의 크기에 따라 다른 자식을 측정하거나 배치해야 하는 경우. SubcomposeLayout을 사용하는 대표적인 이유 중 하나.
- 긴 리스트에서 보이는 항목만 렌더링 하고 필요할 때까지 다른 항목을 지연시키는 등 사용 가능한 공간을 기반으로 항목을 지연 구성 하려는 경우.
- 컴포저블 크기가 컴포지션 타임에 동적으로 정해져야 하는 콘텐츠의 경우
- 레이아웃 로직이 여러 독립적인 단계에 걸쳐 자식을 측정하고 배치해야하는 경우.
- 입력이 변경될 때만 recompose 되어야 하는 LazyList의 헤더와 같은 동적 UI를 처리하는 경우.

<br><br><br>

### 28. Box에 대해서 아는 대로 다 설명해 주세요.

#### Box 동작 방식

- Box는 기본적으로 자식을 왼쪽 상단 정렬로 배열하지만 contentALignment 매개변수를 사용하여 정렬을 커스텀할 수 있음.
- 또한 Modifier 속성을 통해 크기, 패딩, 배경 및 클릭 상호 자굥ㅇ을 커스텀할 수 있음.

<br><br><br>

#### BoxWithConstraints 살펴보기

- BoxWithConstratins는 컴포지션 중 부모의 레이아웃 제약 조건에 대한 접근을 제공하는 고급 레이아웃 API임.
- 일반 Box와 달리 개발자가 사용 가능한 공간 및 크기 제약 조건에 따라 동적으로 UI 결정을 내릴 수 있도록 하여 반응형 디자인 및 적응형 레이아웃에 유용함.

<br><br><br>

#### BoxWithConstraints 작동 방식

- BoxWithConstraints 컴포저블은 컨텐츠 람다 함수 내에서 maxWidth, maxHeight, minWidth, minHeight등과 같은 프로퍼티를 담고 있는 Constraints 스코프를 제공함.
- 해당 값들은 해당 BoxWithConstraints 컴포저블이 랜더링 될 수 있는 크기를 나타내므로 주어진 제약 조건에 따라 자식 UI들을 유동적으로 스타일링 할 수 있음.
- BoxWithConstraints는 내부적으로 SubcomposeLayout으로 되어있기 때문에 불필요한 사용은 최대한 지양해야함.

<br><br><br>

### 29. Arrangement와 Alignment의 차이점에 대해서 설명해주세요.

#### Arrangement란 무엇인가?

- Arrangement는 Row또는 Column과 같이 항목을 단일 방향으로 정렬하는 레이아웃 내에서 여러 자식 컴포저블의 간격 및 분포를 제어함.
- 레이아웃의 주축을 따라 잣기이 어떻게 배치되는 지를 결정함.

<br><br><br>

#### Alignment란 무엇인가?

- Alignment는 자식 컴포저블이 교차축을 따라 부모 내에서 어떻게 배치되는 지를 결정함.

<br><br><br>

### 30. Painter에 대해서 설명해주세요.

#### Painter 작동 방식

- 전통적인 이미지 로딩 접근 방식과 달리 Painter는 이미지 리소스를 표시하는 UI 컴포넌트와 분리되어 있음.
- 이는 Drawable 리소스를 로드하기 위한 PainterResource 또는 벡터 깁나 이미지를 처리하는 VectorPainter를 반환하는 rememberVectorPainter와 같이 다른 이미지 소스로 작업할 때 유용함.
- Compose UI에서 다음과 같은 방식으로 페인터를 구현할 수 있음.
- res/drawable 폴더에서 이미지를 로더하기 위한 painterResource(id).
- 단색으로 영역으로 채우기 위한 ColorPainter(color).
- ImageVector에서 동적으로 VectorPainter를 생성하기 위한 rememberVectorPainter(image = ImageVector).

<br><br><br>

#### 주요 특징

- Painter 객체는 Drawable 리소스를 Compose UI에 렌더링 할 수 있는 형태로 나타내며 안드로이드의 전통적인 Drawable API를 대체하여 사용할 수 있음.
- 이미지나 그래픽이 렌더링되는 방식을 정의할 뿐만 아니라 이를 사용하는 컴포저블의 크기 측정 및 레이아웃에도 영향을 미침.
- 커스텀 페인터를 만드려면 Painter 클래스를 상속받고 커스텀 그래픽 렌더링을 위해 필요한 DrawScope를 제공하는 onDraw 메서드를 구현해야 함.
- 이를 통해 컴포저블 내에서 콘텐츠가 그려지는 방식을 완전히 커스텀할 수 있음.
