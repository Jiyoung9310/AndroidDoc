---
layout: post
title:  "안드로이드 기초 다지기(3) - 인텐트란"
date:   2018-05-29
description: 인텐트가 무엇인지에 대해 알아본다.
---

<p class="intro"><span class="dropcap">안</span>드로이드 기초 다지기 시리즈 세번째</p>

# 안드로이드 기초 - intent란

안드로이드에서는 4대 컴포넌트라고 불리는 구성요소가 있다고 하였다. 이 것들 사이에서 통신할 수 있는 역할을 담당하는 것이 바로 '인텐트'이다.

Intent를 사용하게 되는 경우는 기본적으로 3가지이다.
1. 새로운 Activity를 시작할 때
2. Service를 실행시킬 때
3. BroadCast를 전달할 때

## 어떻게 사용하나?
Intent 객체를 생성하여 컴포넌트에 전달할 데이터나 수행 지시 액션을 담는다. 그리고 Activity, Service 또는 BroadCast 실행 메소드에 intent객체를 넘긴다.
Activity1 -> Activity2를 띄운다고 가정해보면
{% highlight java%}
Intent intent = new Intent(this, Activity2.class);
startActivity(intent);
{% endhighlight %}
위와 같은 코드처럼 사용할 수 있다.
Service를 실행시키고 싶다면 startService(intent)
BroadCast를 전달하고 싶다면 sendBroadcast(intent)
를 사용하면 된다. 아주 쉬움!

## Intent 구성하기
Intent 객체는 정보를 담는 상자이다. 그럼 어떤 정보를 담을 수 있는가?
####1. 컴포넌트 이름
시작할 타깃 컴포넌트 이름을 담는다. 위에 예시에서도 볼 수 있듯이 Intent객체를 생성할 때, 두번째 파라미터에 Activity2.class라는 시작할 컴포넌트 이름을 **명시**하였다.

####2. 액션
수행되어야 하는 작업을 지정해줄 수 있다. Intent객체 안에는 다양한 액션들이 정의되어 있는데 이중에 필요한 것을 가져다가 쓰거나, 사용자가 마음대로 만들어서 지정할 수도 있다.
많이 사용하는 액션으로는
ACTION_VIEW : 데이터를 사용자에게 표시하는 액션
ACTION_CALL : 전화 통화를 시작하는 액션
ACTION_DIAL : 전화번호 표시 화면을 표시하는 액션
정도가 있다.

####3. 데이터
액션 수행에 필요한 데이터를 담는다. 위에서 지정한 액션을 수행하기 위해 부수적으로 필요한 데이터들이 있을 수 있다. 예를 들면 전화를 거는 액션이면 전화번호 데이터를 함께 넘겨줄 수 있다. 데이터는 URI형식으로 넘겨준다.
{% highlight java%}
Intent intent = new Intent(Intent.ACTION_CALL);
intent.setData(Uri.parse("tel:01012341234"))
startActivity(intent);
{% endhighlight %}

####4. 카테고리
수행할 컴포넌트가 어떤 특별한 역할을 맡는다면 거기에 대해서 분류 정보를 지정한다. 예를 들어 CATEGORY_LAUNCHER 속성은 해당 액티비티가 최상위 애플리케이션으로 론처에 나타나야한다는 의미이다.
마찬가지로 Intent객체 내에 다양한 카테고리가 String형태로 정의되어 있어 가져다가 쓰기만 하면 된다.
많이 사용하는 카테고리는
CATEGORY_LAUNCHER : 애플리케이션에서 가장 먼저 실행되는 액티비티
CATEGORY_HOME : 홈 화면을 표시하는 액티비티
CATEGORY_BROWSABLE : 웹 브라우징을 할 수 있는 액티비티
CATEGORY_PREFERENCE : 환경 설정 액티비티
CATEGORY_DEFAULT : 가장 기본이 되는 액티비티

####5. Extra
이외에 작업을 수행하기 위해 더 필요한 데이터가 있다면 extra 데이터로 정보를 넘겨줄 수 있다. extra로 넘기는 데이터는 항상 키-값 쌍을 이루어야 한다. Bundle객체를 생성하여 Intent에 담아 보내도 된다.

####6. Flag
액티비티를 시작할 때 안드로이드 시스템에 처리여부를 알리는 기능을 한다. 사실 제대로 어떻게 활용하는지 모르겠다....

---
지금까지 intent가 무엇이고 이것이 가지는 기능과 역할에 대해서 알아보았다. 정리하자면 다음 다음 컴포넌트가 수행할 동작과 그것에 필요한 정보를 꾹꾹 담아넣을 객체가 intent인 것이다.
