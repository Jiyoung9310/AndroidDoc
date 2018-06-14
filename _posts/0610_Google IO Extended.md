## What's new in Android
- 안드로이드 api 28 개발 완료 - DP3
- 사용자 패턴을 학습하여 자주 사용하는 앱에 따라 가중치, 가중치에 따라 배터리를 관리
- 백그라운드 작업 제한 : 녹음, 캡처, 디바이스 센서 정보 받지 않음
- 구글지도 실내까지도 파악 가능
- Accessibility
- 지문 UI제공 (P부터)
- 노치 UI - display cutout (P부터)
- Ations 앱 노출 - 앱의 특정 기능을 앱을 들어가지 않고도 사용할수 있도록 함
- 노티 안에 이미지도 보낼 수 있도록 개선
- target 26으로 올리기 - 백그라운드 동작 제한 확인 요망
- ----
### P Runtime
-  Dex 컴파일 개선
-  P를 사용하면서 메모리 최적화

### Build
- D8
- 클래스가 만들어지면 자동으로 dex를 만듦 - 빌드속도 개선
- 안스 3.2부터
- 안쓰는 클래스 정리 - proGuard로 했었음 -> R8
- D8 + ProGuard => R8
- android enableR8 = true
- 아직 정식 출시는 안됨
- annotation proesser - 증분 아직 안됨

### App Bundle
- 계속 증가하는 apk 앱 사이즈
- split apk -> App Bundle : apk파일을 생성할 수 있는 bundle을 업로드하면 필요한 디바이스에 따라서 apk를 만들어줌
- 여러 apk를 하나의 파일로
- 트위터 - 앱 사이즈 35% 줄어듦
- 당장 반영할 것
- bundle 만들기 -> aab파일 생성 -> playstore 업로드 -> 기기마다 설치되는 apk를 자동으로 만들어서 사이즈를 측정 -> 배포
- Dynamic Feature = 
- App Bundle + Instant App = 유연한 사용

### JetPack
- viewModel 포함 및 편리한 기능들이 추가되었으니 확인 요망

------
## 새로운 안드로이드 개발 툴
Navigation, WorkManager 훑어보기

### Android Studio 3.2
- jetPack 추가
- app bundle
- reyeclerView
- 빌드 환경
- 등등


- JetPack 라이브러리에 Navigation - 단일 액티비티와 다중 프래그먼트 흐름 제어
- Android X - 기존 라이브러리 묶음, 기존 라이브러리 리팩토링 추가
- Sample Data - UI 에디터 지원, 샘플 데이터 형식 지원
- Slice - P에서 Ation과 연동, 크롬에서 텍스트를 지정하면 app을 사용할 수 있음
- 빌드 지원 - App bundle, D8-기존 컴파일러 대체, R8, CMacker
- Profiler - 에너지 프로파일. 디버깅을 할 때, 현재 디바이스 배터리 확인 가능
- 시스템 트레이스, 프로파일 세션(export, import 가능), 코드레벨 CPU 동작 기록, JNI 레퍼런스 추적 지원

### JetPack 살펴보기
- 기존 기능
	- appCompat
	- multiDex
	- android Test
	- Data Binding
	- Lifecycle
	- Live Data  - room 연동
	- Navigatin*
	- Paging
	- Room
	- ViewModel
	- WorkManager*

### Navigation
- UI 액티비티에 다중 프래그먼트를 사용하기 위함
- Up and Back의 자동 지원
- Bundle 참조 지원
- DeepLink 지원
- 적용 방법 : 그래들 추가 - 매니페스트 리소스(main_nav.xml) 참조 - main_nav.xml 프래스먼트 이동 지정 - 에디터 화면에서 확인
- NaviHostFragment에서 시작
- supportNavigationUp ()- 네비게이션 리소스를 찾는 메소드 추가

### Navigation Controller
- 네비게이션 플로우를 컨트롤 하는 역할
- Resource 파싱 및 관리
- View 에 Tag 저장되어서 관리
- 모든 동작은 런타임에 동작됨

### WorkManager
- intentService, JobScheduler, Firebase JobDisptcher, AlarmManager를 대체
- OS 버전별로 다른 동작에 대한 호환성 제공
- 적용방법 : Worcker상속 worker를 생성


### 동작 이해하기
- workerService 
- 동작 상태 관리를 위한 DB

## Auto, TV, Wear - P 업데이트
### Android Auto
- 메시지
 	- 통합 메시지 및 추가된 액션, 그룹 메시징
- 미디어


### 메시징 앱
- 음성으로 메시지를 확인, 답장
- MessagingStyle 지정

### 미디어 앱
- MediaBrowserService
- MediaSessionService
- 구글 어시스턴트를 통해서 사용하도록 구현 가능
- 어시스턴트를 통해서 검색 가능 - onSearch

### 데스크탑 헤드 유닛
- 차가 없이도 테스트 가능

### Android TV
- content 1st UX


## Review : Material Design from I/O

### New Material Design
- Material Theming
- Gallery
- Material Icons
- New Components

### Material Theming
- Sketch에서 사용가능한 Them Editor
- color, shape, type, icons 플러그인 제공

### Gallery
- 