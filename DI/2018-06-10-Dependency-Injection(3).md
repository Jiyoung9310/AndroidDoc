---
layout: post
title:  "Dagger : Scpoed object graphs(3)"
date:   2018-06-10
description: DI가 무엇인지 알아본다.
---
<p class="intro"><span class="dropcap">안</span>드로이드 아키텍처</p>

# Dagger : Scoped object graphs(3)(번역)

글쓴이 : Antonio Leiva
[이 블로그 글을 번역하였습니다.](https://antonioleiva.com/dagger-3/)

이전 포스팅을 계속 따라오고 있다면, 이미 [dependency injection 기초 1편](http://antonioleiva.com/dependency-injection-android-dagger-part-1/)과 [Dagger fundamentals 2편](http://antonioleiva.com/dagger-android-part-2/)을 읽었을 것이다. 이것은 Dagger와 dependency injection에 대한 마지막 편으로 **scoped object graphs**에 중점을 두고 있다.

## Dagger에서 scoped graphs의 용도는 무엇인가?

우리가 애플리케이션 객체 그래프에서 Dagger 싱글톤을 인스턴스화 시킬 때, app이 destroyed 되기 전까지 싱글톤 객체는 메모리에 살아있다. 하지만 ** 다른 객체가 살아있는 동안 사용가능한 **싱글톤 의존성이 있다. 간단한 예로 view와 그것의 presenter가 있다. 대부분의 경우 동시에 액티비티가 있는 presenter만 사용하게 된다. MVP에서 view가 없는 presenter는 무쓸모다. 액티비티가 destroyed 된 이후엔 그것을 메모리에 상주시킬 필요가 없다.

## scoped graph 생성 방법

간단하다. application graph에 추가하면 된다. 작성자는 액티비티 당 모듈 하나를 작성하였다. LoginActivity 예제를 살펴보자.
~~~
@Module (

	injects = LoginActivity.class,
    addsTo = AppModule.class
)
public class LoginModule {

	private LoginView view;
    public LoginModule(LoginView view) {
    	this.view = view;
    }
    @Provides @Singleton public LoginView provideView() {
    	return view;
    }
    @Provides @Singleton
    public LoginPresenter providePresenter(LoginView loginView, LoginInteractor loginInteractor) {
    	return new LoginPresenterImpl(loginView, loginInteractor);
    }
}
~~~
이 모듈은 LoginActivity를 주입한다, 왜냐하면 생성자를 통해서가 아닌 activity에 직접 presenter를 주입해야하기 때문이다. 그것은 activity graph가 생성될 떄 AppModule에 추가될 것이다. 둘다 @Module 어노테이션으로 선언해야한다.

보다시피, LoginPresenter에 새로 의존성을 추가했다. LoginInteractor는 InteractorsModule이라는 새로운 모듈로에서 주입된다. 간단하고 기초적인 모듈이다:
~~~
@Module(

	library = true
)
public class InteractorsModule{

	@Provides public FindItemsInteractor provideFindItemsInteractor() {
    	return new FindItemsInteractorImpl();
    }
    @Provides public LoginInteractor provideLoginInteractor() {
    	return new LoginInteractorImpl();
    }
}
~~~
AppModule에 이전에 있던 DomainModule과 함께 추가한다:
~~~
@Module(

	injects = {
    	App.class
    },
    includes = {
    	DomainModule.class,
        InteractorsModule.class
    }
)
public class AppModule {
	...
}
~~~

object graph를 생성하자. application graph의 plus()를 호출하고 새 모듈을 전달하여 생성이 가능하다. 그리하여 App에 이 메소드를 만들었다.
~~~
public ObjectGraph createScopedGraph(Object... modules) {

	return objectGraph.plus(modules);
}
~~~
이제 Activity에서 인스턴스화 하는 대신 presenter를 주입하여 그래프를 만들고 activity를 삽입한다.
~~~
public class LoginActivity extends Activity implements LoginView, View.OnClickListener {

	@Inject LoginPresenter presenter;
    ...
    private ObjectGraph activityGraph;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
    	super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        ...
        activityGraph = ((App) getApplication()).createScopedGraph(new LoginModule(this));
        activityGraph.inject(this);
    }
    @Override protected void onDestroy() {
    	super.onDestroy();
        activityGraph = null;
    }
    ...
}
~~~
onDestroy()에서 null을 셋팅함으로써 최대한 빨리 가비지 콜렉터로부터 자유롭게 한다. 이로써 Dagger를 사용해 presenter, view 그리고 model을 주입하였다.

## 결론
Dagger는 아주 강력한 도구이다. 하지만 러닝커브가 다른 라이브러리들에 비해 크다. 왜냐하면 주요 개념을 받아들이고 그것이 왜 유용한지 이해하는 것이 중요하기 때문이다. 이 글이, 처음 시작할 때 발견하게 될 어려움을 해결하는데에 도움이 되었으면 한다.

scoped graph를 인스턴스화 하는 반복 작업을 캡슐화하는 BaseActivity를 만들어 코드를 업데이트 하였다.

[Dagger example at Github](https://github.com/antoniolg/DaggerExample)

의구심이 드는 부분이 있거나 다른 포스트가 개념을 정의하는데 유용하다고 생각한다면 댓글을 남겨달라.
