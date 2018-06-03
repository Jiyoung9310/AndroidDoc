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

이전 포스팅을 읽었다면, 아마 당신은 실제 코드를 보고싶어 할 것이다. [Dagger page에 커피 메이커](https://github.com/square/dagger/tree/master/examples/simple)에 관한 아름다운 예제들이 있다. 그리고 좀 경험이 있는 사람들을 위한 [Jake Wharton의 model project](https://github.com/JakeWharton/u2020)와 같은 멋진 예제도 있다. 하지만 우린 좀더 쉬운 것이 필요하다. 커피는 우리의 메인 비즈니스 모델도 아니다. 그래서 이 글에서는 주입이 어디에서 이루어지는지 기초적인 이해를 위한 간단한 컴포넌트 예제를 제공할 것이다.

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
첫번째는 Dagger 라이브러리, 두번째는 Dagger 컴파일러이다. 그것은 의존성 주입을 할수 있도록 필요한 클래스를 생성할 것이다. 이것이 전처리된 클래스를 생성함으로써 대부분의 리플렉션을 피할 수 있는 방법이다. 그것은 프로젝트를 컴파일하기 위해서만 필요하고 앱에서 사용하지 않기 때문에, 'provided'이라고 표시하여 최종 apk에 포함되지 않도록 한다.

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
---
**새로 나온 용어 설명**
- @Module : 이 클래스를 Dagger 모듈로 정의
- injects : 이 모듈이 의존성을 주입할 클래스. 객체에 직접 주입되는 클래스를 지정해야 한다. 이 내용은 곧 다룰 예정이다.
- @Provides : injection provider 메소드를 정의. 메소드 이름은 전혀 중요하지 않으며, 제공되는 클래스 유형만 다룬다.
- @Singleton : 인스턴스가 존재한다면 항상 동일한 객체의 인스턴스를 리턴하며 이는 일반적인 싱글톤보다 훨 낫다. 존재하지 않는다면, 매번 유형이 삽입될 때마다 새로운 인스턴스가 생성된다. 새 인스턴스를 만들지 않고 기존 인스턴스를 반환하는 경우에 싱글톤을 주석으로 추가하지 않으면 provider가 무엇을 하는지 더 잘 설명할 수 있다. Application 인스턴스는 고유하다.
---


>왜 일반 싱글톤이 나쁜가
>싱글톤은 아마 프로젝트가 가질 수 있는 가장 위험한 의존성이다. 첫째로, 인스턴스를 생성하지 않았다는 가정 하에, 우리가 어디서 그것을 사용했는지 알기 어렵기때문에 이것은 "숨겨진 의존성"이기 때문이다. 두번째로는 그것을 다른 모듈에서 테스트하거나 대체할 mock 방법이 없다. 코드를 유지, 테스트, 고도화하기 어려워진다. 반대로, 주입된 싱글톤은 싱글톤(고유한 인스턴스)으로써의 장점을 가지면서 언제든지 새 인스턴스를 만들 수 있으므로, mock하고 다른 코드로 대체하거나 하위 클래스로 만들거나 공통 인터페이스 구현을 하는 것이 더 쉽다.


우리는 새 패키지에 domain이라는 이름의 모듈을 생성할 것이다. 모든 아키텍처 계층에 (최소한) 모듈을 포함하는 것이 매우 유용하다. 이 모듈은 AnalyticsManager를 제공하는데, 앱이 시작될 때 이벤트를 던지며 토스트를 표시한다. 실제 프로젝트에서 이 관리자는 구글 애널리틱스와 같은 분석 서비스를 호출한다.
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
이 모듈이 완성되지 않은 것을 인지함으로써(complete값이 false), 우리는 이 모듈의 일부 의존성은 다른 모듈에서 제공되어야 한다고 말한다. AppModule에서 온 Application 같은 경우이다. DI로부터 이 AnalyticsManager를 요구할 때, Dagger는 이 메소드를 사용할 것이다. 다른 의존성인 Application이 객체 그래프에 요청될 것임을 탐지할 것이다. 또한 우리는 이 모듈을 라이브러리로 지정해야한다. 왜냐하면 Dagger 컴파일러가 AnalyticsManager가 자체 또는 주입된 클래스에서 사용되지 않았음을 감지하기 때문이다. 이것은 AppModule의 라이브러리 모듈로 작동한다.

우리는 AppModule이 여기에 포함되도록 지정할 것이다. 이전 클래스도 돌아가보면,
~~~
@Module(
	injects = {
    	App.class
    },
    includes = {
    	DomainModule.class
    }
)
public class AppModule {
...
}
~~~
includes 속성은 저런 목적을 위해 존재한다.

## 객체 그래프 생성하기
객체 그래프는 모든 의존성이 살아있는 공간이다. 객체 그래프는 생성된 인스턴스를 담고 있다. 그리고 우리가 추가한 객체에 그것을 주입할 수 있다.

이전 예제(AnalyticsManager)에서 우리는 생성자를 통해 주입되는 "고전적인" 의존성 주입을 살펴보았다. 하지만 우리가 생성자로 제어할 수 없는 안드로이드 클래스들(Application, Activity)이 존재한다. 그러므로 간단하게 의존성을 주입하는 방법이 필요하다.

ObjectGraph생성과 이 직접 주입의 콤비네이션은 App 클래스에서 표현된다. 메인 객체 그래프는 Application 클래스에서 생성되고 의존성을 갖기 위해서 주입된다.
~~~
public class App extends Application {
	private ObjectGraph objectGraph;
    @Inject AnalyticsManager analyticsManager;
    
    @Override public void onCreate() {
    	super.onCreate();
        objectGraph = ObjectGraph.create(getModules().toArray());
        objectGraph.inject(this);
        analyticsManager.registerAppEnter();
    }
    
    private List<Object> getModules() {
    	return Arrays.<Object>asList(new AppModule(this));
    }
}
~~~
@Inject 어노테이션을 통해 의존성의 정의한다. 필드는 반드시 public이나 디폴트 범위이어야 Dagger가 할당할 수 있다. 우리는 모듈을 가진 배열을 생성한다. (우리는 단 하나만을 가지고 있다. DomainModule은 AppModule에 포함됨.) 그리고 그걸로 ObjectGraph를 생성한다. 그 후, App인스턴스를 수동으로 삽입한다. 호출 후엔 의존성이 주입되며, AnalyticsManager 메소드를 호출할 수 있게 된다.

## 결론
이제 우리는 Dagger에 대해서 기본적으로 알게 되었다. ObjectGraph와 Modules는 Dagger를 효율적으로 사용하기 위해서 반드시 습득해야하는 가장 인상적인 컴포넌트이다. [Dagger site](http://square.github.io/dagger/)에 설명되어있는 lazy injections이나 provider injection과 같은 더 많은 툴들이 있지만, 우리가 살펴본 것들을 잘 사용할 수 있기 전까지는 보는 것을 추천하지 않는다.

소스코드가 [Github](https://github.com/antoniolg/DaggerExample)에 있다는 사실을 잊지 마라.

다음(아마 마지막) Dagger에 관한 포스트에서는 [scoped object graphs](https://antonioleiva.com/dagger-3/)에 대해서 다룰 것이다. 기본적으로 생성자가 있는 곳에만 존재하는 새로운 객체 그래프를 만드는 것으로 구성된다. Activity에 대한 scoped graph를 만드는 것이 일반적이다.

[RSS](https://antonioleiva.com/feed/) 또는 [Google+ 프로필](https://plus.google.com/+AntonioLeivaGordillo)을 통해 지켜봐