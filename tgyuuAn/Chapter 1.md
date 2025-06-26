Q1.) Intent란 무엇인가요?

	⁃	수행될 작업에 대한 추상적인 설명
	⁃	Activity, Service, BroadcastReceiver가 통신할 수 있도록 하는 메시징 객체 역할을 함
	⁃	Intent는 일반적으로 Activity를 시작하거나, Broadcast를 보내거나, Service를 시작하는 데 사용함
	⁃	컴포넌트 간에 데이터를 전달할 수 있어 안드로이드의 시스템에서 근본이 되는 요소임
	⁃	안드로이드는 명시적, 암시적 두 가지 유형의 Intent가 있음

명시적 Intent

	⁃	정의 : 명시적 Intent는 호출할 컴포넌트를 직접 이름으로 지정하여 정확히 명시
	⁃	사용 사례 : 명시적 Intent는 대상 컴포넌트를 알고 있을 대 사용
	⁃	시나리오 : 동일한 앱 내에서 한 Activity에서 다른 Activity로 전환하는 경우 명시적 Intent를 사용

암시적 Intent

	⁃	정의 : 암시적 Intent는 특정 컴포넌트를 지정하지 않고 수행할 일반적인 작업을 선언. 시스템은 액션, 카테고리, 데이터를 기반으로 어떤 컴포넌트가 Intent를 처리할 수 있는 지 결정.
	⁃	사용 사례 : 암시적 Intent는 다른 앱이나 시스템 컴포넌트가 처리할 수 있는 작업을 수행하려 할 때 유용
	⁃	시나리오 : 브라우저에서 웹 페이지를 열거나 다른 앱과 콘텐츠를 공유하는 경우 암시적 Intent를 사용. 시스템이 Intent를 처리할 앱을 결정


명시적 Intent는 대상 컴포넌트가 잘 알려진 앱 내부 네비게이션에서 주로 사용.
암시적 Intent는 대상 컴포넌트를 직접 알 수는 없지만 액션 등으로 외부 앱이나 컴포넌트가 처리할 수 있는 작업에 사용


Q : 명시적 인텐트와 암시적 인텐트의 차이점은 무엇이며, 각각 어떤 시나리오에서 사용해야 하나요?
A : 명시적 인텐트는 대상 컴포넌트를 잘 알고 있는 경우, 줄 내부 앱에서 Activity의 이동과 같은 네비게이션에서 주로 사용되고, 암시적 인텐트는 외부 앱이나 외부 컴포넌트를 사용할 때 액션이나 카테고리 , 데이터 등을 기반으로 작업을 수행하려고 할 때 사용

Q : 안드로이드 시스템은 암시적 인텐트를 처리할 앱을 어떻게 결정하며, 적합한 어플리케이션을 찾지 못하면 어떻게 되나요 ?
A : 암시적 인텐트는 작업을 실행할 수 있는 기기의 앱을 호출할 수 있는 작업을 지정. 시스템은 설치된 모든 앱을 검사하여 이러한 종류의 인텐트를 처리할 수 있는 앱을 결정. 처리할 수 있는 앱이 하나 뿐이라면 앱이 바로 열리지만, 하나도 없다면 ActivityNotFouneException을 받음. 여러 활동이 인텐트를 받을 수 있을 경우, 시스템은 사용자가 사용할 앱을 선택할 수 있도록 대화 상자를 표시함.


인텐트 필터(Intent Filters)란?

	⁃	앱 컴포넌트가 링크 열기나 브로드캐스트 처리와 같은 특정 Intent에 어떻게 응답할 수 있는지를 정의
	⁃	Activity, Service, BroadcastReceiver가 처리할 수 있는 Intent 유형을 선언하는 필터 역할을 함. AndroidManifest.xml 파일에 명시됨.
	⁃	각 Intent filter는 들어오는 Intent와 정확히 일치시키기 위해 액션, 카테고리 및 데이터 유형을 포함할 수 있음.
	⁃	Intent filter를 적절하게 정의하면 앱이 다른 앱 및 시스템 컴포넌트와 원활하게 상호 작용하여 기능을 향상시킬 수 있음.





Q2.) PendingIntent의 목적은 무엇인가요?

	⁃	다른 어플리케이션이나 시스템 컴포넌트가 어플리케이션을 대신하여 미리 정의된 Intent를 나중에 실행할 수 있는 권한을 부여하는 또 다른 종류의 Intent. 알림이나 서비스와의 상호작용과 같이 앱의 수명 주기를 벗어나 트리거되어야 하는 작업에 특히 유용
	⁃	특징 : 일반 Intent의 래퍼 역할을 하며 앱의 생명 주기를 넘어서 지속될 수 있도록 함. 다른 앱이나 시스템 서비스에 Intent 실행을 위임함. PendingIntent는 Activity, Service 또는 BroadcastReceiver를 위해 생성될 수 있음

PendingIntent는 동작 방식과 시스템 또는 다른 컴포넌트와의 상호 작용 방식을 제어하는 다양한 플래그를 지원

	⁃	FLAG_UPDATE_CURRENT : 기존 PendingIntent를 새 데이터로 업데이트 함
	⁃	FLAG_CANCEL_CURRENT : 새 PendingIntent를 만들기 전에 기존 PendingIntent를 취소함
	⁃	FLAG_IMMUTABLE : PendingIntent를 변경 불가능하게 만들어 수신자가 수정하는 것을 방지함
	⁃	FLAG_ONE_SHOT : PendingIntent가 한 번만 사용될 수 있돌고 보장합니다.

Notification, Alarm, Service 등에서 사용, 안드로이드 12이상에서는 악의적인 앱이 기존 Intent를 수정하는 것을 방지하기 위해 FLAG_IMMUTABLE로 설정

PendingIntent는 앱이 활성 상태가 아닐 때에도 앱과 시스템 컴포넌트 또는 다른 앱 간의 원활한 통신을 가능하게 하는 안드로이드의 핵심 메커니즘. 플래그와 권한을 신중하게 관리함으로써 지연된 작업의 안전하게 효율적인 실행을 보장할 수 있음.

Q : PendingIntent란 무엇이며 일반 Intent와는 어떻게 다른가요? PendingIntent 사용이 필요한 시나리오를 제시해 줄 수 있나요 ?
A : PendingIntent는 Intent를 래핑한 래퍼 클래스. 주로 외부 앱이나 다른 컴포넌트에서 실행될 Intent를 미리 만들어서 보냄으로써 Intent의 실행을 다른곳으로 위임. 즉, Intent는 미리 만들어 놓고 특정한 트리거에 의해서 뒤늦게 실행될 수 있도록 하는 래퍼 클래스



Q3.) Serilizable과 Parcelabe의 차이점은 무엇인가요?

안드로이드에서 Serializable과 Parcelabe은 모두 다른 컴포넌트 간에 데이터를 전달하는 데 사용되는 메커니즘.

Serializable 특징
	⁃	Java 표준 인터페이스 : 객체를 바이트 스트림으로 변환하여 Activity간에 전달하거나 디스크에 쓸 수 있도록 하는 표준 Java 인터페이스
	⁃	리플렉션 기반 : Java 리플렉션을 통해 작동. 시스템이 런타임에 클레스와 필드를 동적으로 검사하여 객체를 직렬화
	⁃	성능 : Serializabl은 리플렉션이 느린 프로세스이기 때문에 Parcelable에 비해 느림. 또한, 직렬화 중에 많은 임시 객체를 생성하여 메모리 오버헤드를 증가시킴
	⁃	사용 사례 : Serializable은 성능이 중요하지 않거나 안드로이드 특정 코드가 아닌 코드베이스를 다룰 때 유용


Parcelable 특징
	⁃	Android 기반 인터페이스 : 안드로이드 컴포넌트 내에서 고성능 IPC를 위해 특별히 설계된 안드로이드 특정 인터페이스
	⁃	성능 : Parcelable은 안드로이드에 최적화 되어있고, 리플렉션이 아닌 직렬화와 역직렬화에 의존하므로 더 빠름
	⁃	사용 사례 : Parcelable은 성능이 중요하거나 안드로이드 데이터 전달, 특히 IPC나 Activity, Service 간 데이터 전달에 선호

최근 안드로이드 개발에서는 kotlin-parcelize plugin이 구현을 자동으로 생성하여 Parcelable 객체를 만드는 과정을 단순화 함.
이 접근 방식은 이전에 수동 메커니즘에 비해 더 효율저임. 클래스에 Parcelize 어노테이션을 붙이기만 하면 필요한 구현을 생성함



Q4.) Context란 무엇이며 어떤 유형의 Context가 있나요?

Context는 애플리케이션의 환경 또는 상태를 나타내며 어플리케이션별 리소스 및 클래스에 대한 접근을 제공. 앱과 안드로이드 시스템간의 브릿지 역할을 하여 컴포넌트가 리소스, 데이터베이스, 시스템 서비스 등에 접근할 수 있도록 함. Context는 Activity 실행, 애셋 접근 또는 레이아웃 인플레이션과 같은 작업에 필수적인 컴포넌트.


Application Context 
	⁃	Application Context는 애플리케이션의 라이프 사이클과 연결되어 있음.
	⁃	현재 Activity나 Fragment와 독립적인 전역적이고 오래 지속되는 Context가 필요할 때 사용됨. 이 Context는 getApplicationContext()를 호출하여 휙득할 수 있음.

사용 사례 : 
	⁃	ApplicationContext는 SharedPreference나 데이터베이스와 같은 애플리케이션 전체 리소에 접근하는 경우
	⁃	전체 앱 생명주기 동안 지속되어야 하는 BroadcastReceiver를 등록하는 경우
	⁃	앱 생명주기 동안 유지되는 라이브러리나 컴포넌트를 초기화하는 경우


Activity Context
	⁃	Activity의 생명주기와 연결되어 있음. Activity에 특정한 리소스 접근, 다른 Activity 시작, 레이아웃 인플레이션에 사용됨.

사용 사례 :
	⁃	UI 컴포넌트를 생성 또는 업데이트하는 경우
	⁃	다른 Activity를 실행하는 경우 (Application Context가 아닌 Activity Context로 해야함. 백스택 관리의 이유와 Activity별 개별 테마가 반영되지 않음)
	⁃	현재 Activaty 범위에 있는 리소스나 테마에 접근하는 경우


Service Context
	⁃	Service Context는 Service의 생명주기와 연결되어 있음. 주로 네트워크 작업 수행이나 음악 재생과 같은 백그라운드에서 실행되는 작업에 사용됨. Service에 필요한 시스템 수준 서비스에 대한 접근을 제공함


Broadcast Context
	⁃	Broadcast Context는 BroadcastReceiver가 호출될 때 제공됨. 이는 수명이 짧으며 일반적으로 특정 브로드캐스테 응답하는 데 사용됨. 따라서 Broadcast Context로 장기적인 태스크를 수행하면 안됨


Context의 일반적인 사용 사례
	⁃	리소스 접근 : getString() 또는 getDrawable()과 같은 메서드를 사용
	⁃	레이아웃 인플레이션 : LayoutInflater를 사용하여 XML 레이아웃을 뷰로 인플레이션하는 데 Context를 사용
	⁃	액티비티 및 서비스 시작 : Activity와 Service를 시작하려면 Context가 필요함
	⁃	시스템 서비스 접근 : Context는 getSystemService()를 통해 ClipboardManager 또는 ConnectivityManager와 같은 시스템 수준 서비스에 대한 접근을 제공함
	⁃	데이터베이스 및 SharedPreference 접근 : SQLite 데이터베이스나 SharedPreference와 같은 영구 저장 메커니즘에 접근하는 데 Context를 사용함

Q : 안드로이드 애플리케이션에서 올바른 유형의 Context를 사용하는 것이 왜 중요하며, Activity Context에 대해 오랜 참조를 유지하는 것은 잠재적으로 어떤 문제를 발생시킬 수 있나요 ?
A : 올바른 유형의 Context를 사용하지 않을 경우 해당 컴포넌트의 생명 주기 보다 Context가 더 오래 살아있게 되어 GC가 제때 수행되지 못하고 메모리 누수가 발생할 수 있다. 백그라운드 스레드에서 UI와 관련된 작업을 하게 될 경우 Race Condition과 같은 문제가 생길 수 있으므로 Ui 작업시 에러를 내며 크래시가 날 수 있다. 이 떄는 반드시 메인 스레드로 전환한 후 작업을 해야한다./

ViewModel에서 Aplication Context가 아니라 Activity Context를 참조하는 등 Activity Context에 대해 오랜 참조를 유지하면 Activity가 소멸되었을 때 해당 Context는 소멸되지 않고 Activity  인스턴스를 계속 잡아두고 있어 GC가 제때 일어나지 않아 메모리 누수가 발생할 수 있다.


Context Wrapper란?
	⁃	ContextWrapper는 Context를 상속받고 있는 클래스로, Context 객체를 감싸서 래핑된 Context에 대한 호출을 위임하는 기능을 제공. 원본 Context의 동작을 ㅅ줭하거나 확장하기 위한 중간 계층 역할을 함. ContextWrapper를 사용하면 Context와 직접적인 소통을 하지 않고도 특정 기능을 커스텀할 수 있음.
