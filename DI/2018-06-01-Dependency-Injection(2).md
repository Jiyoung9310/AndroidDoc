---
layout: post
title:  "Android에서의 Dependency injection : Dagger(2)"
date:   2018-06-01
description: DI가 무엇인지 알아본다.
---
<p class="intro"><span class="dropcap">안</span>드로이드 아키텍처</p>

# Android에서의 Dependency injection : Dagger(2)(번역)

글쓴이 : Antonio Leiva
[이 블로그 글을 번역하였습니다.](https://antonioleiva.com/dagger-android-part-2/)

이전 포스팅을 읽었다면, 아마 당신은 실제 코드를 보고싶어 할 것이다. [Dagger page에 커피 메이커](https://github.com/square/dagger/tree/master/examples/simple)에 관한 아름다운 예제들이 있다. 그리고 좀 경험이 있는 사람들을 위한 [Jake Wharton의 model project]와 같은 멋진 예제도 있다. 하지만 우린 좀더 쉬운 것이 필요하다. 커피는 우리의 메인 비즈니스 모델도 아니다. 그래서 이 글에서는 주입이 어디에서 이루어지는지 기초적인 이해를 위한 간단한 컴포넌트 예제를 제공할 것이다.

소스 코드는 깃허브 [DaggerExample repository](https://github.com/antoniolg/DaggerExample)에 설명되어있다.

## 프로젝트에 Dagger 포함시키기
Dagger를 사용하기 위해선 두개의 라이브러리가 추가되어야 한다.
~~~
dependencies {
	compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.squareup.dagger:dagger:1.2.+'
    provided 'com.squareup.dagger:dagger-compiler:1.2.+'
}
~~~
첫번째는 Dagger 라이브러리, 두번째는 Dagger 컴파일러이다. 그것은 의존성 주입을 할수 있도록 필요한 클래스를 생성할 것이다. 이것이 전처리된 클래스를 생성함으로써 대부분의 리플렉션을 피할 수 있는 방법이다. 그것은 프로젝트를 컴파일하기 위해서만 필요하고 앱에서 사용하지 않기 때문에, '제공된'이라고 표시하여 최종 apk에 포함되지 않도록 한다.

## 첫 모듈 만들기
모듈은 Dagger와 함께 일상처럼 접하게 될테니 익숙해져야된다. 모듈은 우리가 주입할 객체의 인스턴스를 제공하는 클래스이다. 그것은 @Module 어노테이션으로 정의된다. 몇가지 추가 매개변수도 있는데 우리가 그것을 언제 사용하는지 설명하겠다.

예를 들어서, Application Context를 제공하는 AppModule이라는 클래스를 생성한다. 일반적으로 쉽게 접근할 수 있다. Application을 상속받는 App을 생성하고 manifest에 추가한다.
~~~
@Module(
	injects = {
    	App.class
    }
)
public class AppModule {
	private App app;
    public AppModule(App app) {
    	this.app = app;
    }
    @Provides @Singleton public Context provideApplicationContext() {
    	return app;
    }
}
~~~
@Module : 이 클래스를 Dagger 모듈로 정의
injects : 이 모듈이 의존성을 주입할 클래스. 객체에 직접 주입되는 클래스를 지정해야 한다. 이 내용은 곧 다룰 예정이다.
@Provides : injection provider 메소드를 정의. 메소드 이름은 전혀 중요하지 않으며, 제공되는 클래스 유형만 다룬다.
@Singleton : 인스턴스가 존재한다면 항상 동일한 객체의 인스턴스를 리턴하며 이는 일반적인 싱글톤보다 훨 낫다. 존재하지 않는다면, 매번 유형이 삽입될 때마다 새로운 인스턴스가 생성된다. 새 인스턴스를 만들지 않고 기존 인스턴스를 반환하는 경우에 싱글톤을 주석으로 추가하지 않으면 provider가 무엇을 하는지 더 잘 설명할 수 있다. Application 인스턴스는 고유하다.

- **왜 일반 싱글톤이 나쁜가**
싱글톤은 아마 프로젝트가 가질 수 있는 가장 위험한 의존성이다. 첫째로, 우리가 인스턴스를 생성하지 않았다는 가정 하에, 우리가 어디서 그것을 사용했는지 알기 어렵기때문에 이것은 "숨겨진 의존성"이기 때문이다. 두번째로는 그것을 다른 모듈에서 테스트하거나 대체할 mock 방법이 없다. 코드는 유지, 테스트, 고도화하기 어려워진다. 반대로, 주입된 싱글톤은 싱글톤(고유한 인스턴스)으로써의 장점을 가지면서 언제든지 새 인스턴스를 만들 수 있으므로, mock하고 다른 코드로 대체하거나 하위 클래스로 만들거나 공통 인터페이스 구현을 하는 것이 더 쉽다.

우리는 새 패키지에 domain이라는 모듈을 생성할 것이다. 모든 아키텍처 계층에 (최소한) 모듈을 포함하는 것이 매우 유용하다. 이 모듈은 애널리틱스 매니저를 제공하는데, 앱이 시작될 때 이벤트를 던지며 토스트를 표시한다. 실제 프로젝트에서 이 관리자는 구글 애널리틱스와 같은 분석 서비스를 호출한다.
~~~
@Module(
	complete = false,
    library = true
)
public class DomainModule {
	@Provides @Singleton public AnalyticsManager provideAnalyticsManager(Application app) {
    return new AnalyticsManager(app);
    }
}
~~~