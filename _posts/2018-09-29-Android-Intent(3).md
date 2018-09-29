---
layout: post
title:  "안드로이드 기초 다지기(5) - 인텐트 필터란"
date:   2018-09-29
description: 인텐트 필터에 대해 알아본다.
---

<p class="intro"><span class="dropcap">안</span>드로이드 기초 다지기 시리즈 다섯번째</p>

# 안드로이드 기초 - intent filter

암시적 인텐트로 컴포넌트를 호출할 때는 해당 컴포넌트의 이름이 아니라 "액션"을 통해서 호출한다고 하였다. 
intent filter에 그 액션이 지정된 컴포넌트를 모두 호출하는 것이다. 그렇다면 intent filter를 컴포넌트에 어떻게 설정하는지 알아보자.

각 intent filter는 앱의 매니페스트 파일에 있는 <intent-filter> 태그로 정의할 수 있다.
<intent-filter> 내부에서는 다음과 같은 3가지 요소 중 하나 이상을 사용하여 허용할 인텐트 유형을 지정할 수 있다.

1. 액션 action
2. 데이터 data
3. 카테고리 category

예시 코드를 먼저 살펴보자.
~~~
<activity android:name="ShareActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
</activity>
~~~
```ShareActivity```라는 액티비티 컴포넌트 안에 <intent-filter>를 정의하였다.
액션의 name 특성에서 이 컴포넌트에서 허용된 인텐트 작업을 선언하였다. 
만약 암시적 인텐트로 ```Intent intent = new Intent("android.intent.action.SEN"D)```를 호출한다면 위의 ```ShareActivity```가 불려질 것이다.
데이터는 허용된 데이터 유형을 선언하며, 데이터 URI(scheme, host, port, path 등)나 MIME 유형 등의 속성을 정의할 수 있다.
카테고리는 name 특성에서 허용된 인텐트 카테고리를 선언한다. -> [인텐트 설명 참조](https://github.com/Jiyoung9310/AndroidDoc/blob/master/_posts/2018-05-29-Android-Intent.md)
