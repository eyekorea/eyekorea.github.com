---
layout: post
title:  "checkbox, radio 요소 check 시 fixed element 오작동 버그"
date:   2015-07-15 10:18:00
categories: issue
description : "IOS 7, 8 버전에서 발생하는 버그, checkbox, radio 요소가 fixed elements 에 미치는 버그"
---

# IOS safari 브라우저의 버그
상단이든 하단이든 혹은 촤측이든... 어디든 아직까지 모든 모바일 브라우저가 안정적이지 않다.  
[지난 포스팅](http://eyekorea.github.io/issue/2015/06/15/foot-fixed-form-Issues-in-ios-safari.html)에 이어 이번에도 
mobile safari 의 문제이다. 상황도 비슷한 것이 상단, 하단에 각각 고정되는 element 가 있고 화면 중앙에 컨텐츠 영역이 있다.  
{% highlight html %}
<style>
	#topNav{
		min-width: 320px;
		width:100%;
		height:60px;
		position: fixed;
		left: 0;
		top: 0;
	}
	#contents{
		min-width: 320px;
		width:100%;
		padding:60px 0 100px;
	}
	#footNav{
		min-width: 320px;
		width:100%;
		height: 100px;
		position: fixed;
		bottom: 0;
		left: 0;
	}
</style>
<!-- 생략... -->
<div id="topNav">상단 고정영역</div>
<div id="contents">컨텐츠 영역</div>
<div id="footNav">하단 고정영역</div>
{% endhighlight %}

대략적으로 코드는 위와 같이 구분할 수 있을 듯 하다. 이번 문제는 위와 같이 `position:fixed` 스타일을 갖는 요소들이 있는 상태에서 
`input[type="radio"]` 나 `input[type="checkbox"]` 가 문서 어디에든 존재할경우에 발생한다. 문서에 존재하는 것 자체는 문제가 아니고, 
**해당 요소를 체크하거나 클릭하여 값이 변경되었을때** 발생한다.  
고정되어 있는 요소가 화면에서 사라지는데, fixed 속성이 아닐경우의 자기 위치로 돌아간다. 동작도 산발적이어서 어떤때는 고정되어 있고 어떤때는 사라진다.  
현재 포스팅 하고 있는 시점에서 IOS 8.4 버전이 업데이트 되었으나 아직 테스트를 진행하지는 않았다.

# 해결방법
결론부터 말하자면 *기본 form 요소를 사용하고자 할때 해결방법이 없다.*  
해결을 위해서는 checkbox 혹은 radio 요소가 화면에 노출되지 않아야 한다.(`display:none` or `visibility:hidden`)
나는 checkbox 에 디자인 요소가 추가되어 label 태그의 :before, :after 요소를 이용해 기존 checkbox 를 덮는 방식으로 구현을했다. 따라서 input 
요소를 숨기더라도 크게 문제는 없었다.

# 마치며
*작은 화면에 무언가 고정시키는 것* 에 대해 나는 하고싶은 말이 많다. 내 개인적인 생각은 나중으로 미루고 해당 버그가 빨리 패치되길 바란다.

