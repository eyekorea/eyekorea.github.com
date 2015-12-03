---
layout: post
title:  "모바일 웹 브라우저에서의 iframe 사용 이슈"
date:   2015-11-27 18:00:00
categories: issue
description : "모바일 웹 브라우저에서 iframe 사용은 자제해야 한다. 그렇지 않으면 굉장히 피곤해 진다."
---

# 모바일에서 iframe ?
iframe 은 현재 화면에서 도메인에 상관 없이 다른 페이지를 보여주어야 할때 유용하다. 이점은 인정... 하지만 모바일에서는 조금 다르다. 나는 최근에 내가 겪어야만 했던 모바일 브라우저에서의 iframe 문제에 대해 몇가지 예를 들어 정리 해 보겠다.

## iframe 내부에서 중앙에 떠야 하는 레이어.
- 사실 아래와 같은 마크업이라면 스크립트와 관계 없이 화면 중앙에 레이어를 띄울 수 있다.

{% highlight html %}
<div id="wrap">
	<div class="contents">컨텐츠 영역</div>
	<div class="layer_wrap">
		<div class="layer_block">
			<p>미들 정렬 레이어 영역.</p>
			<p>해당 영역은 스크롤과 관계 없이 화면 중앙에 위치 할 수 있다.</p>
		</div>
	</div>
</div>
{% endhighlight %}

{% highlight css %}
/* 주의 : 해당 샘플 코드는 ms 계열의 프리픽서는 제외되어 있다. */
.layer_wrap{
	z-index: 100;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: -webkit-box;
    display: -webkit-flex;
    display: flex;
    -webkit-align-items: center;
    -webkit-box-align: center;
    align-items: center;
}
.layer_block{
    background-color:#eee;
}
{% endhighlight %}

- 위 코드의 샘플화면은 [이곳](/htm/iframe-test/test_layer.html)에서 확인 할 수 있다.(모바일 뷰에서 확인을 권장한다.)
- 코드상의 문제는 없다. 다만 위의 코드가 iframe 내에서 동작한다면 문제가 발생한다. 위의 페이지를 iframe 에서 불러왔다. 코드 확인은 [이곳](/htm/iframe-test/test.html)에서 할 수 있다. PC 에서 개발자 모드로 모바일 뷰를 확인하거나 혹은 안드로이드를 사용하는 사람은 해당 문제를 알 수 없다. 안드로이드는 4버전대 이상만 테스트 했으나, 문제는 없었다. 문제는 **IOS**에서만 발생하였는데 아래 화면은 `Xcode`의 `simulator` 에서 일반페이지와 iframe 일경우를 각각 스크롤 없이 캡쳐한 화면이다.
![ios 캡쳐 이미지](/htm/iframe-test/img/ios_cap.png)

- 문제 발생이유는 간단하다. iframe 내에서는 window 의 height 를 페이지 전체로 인식하였기 때문... 아래 테스트를 보면 간단히 알 수 있다.
{% highlight js %}
// test 를 위해 test_layer.html 안에 아래 스크립트 코드를 삽입했다.
// 문서가 열리면 윈도우 높이를 경고창에 띄운다.
    $(function(){
        alert($(window).height());
    });
{% endhighlight %}
![ios 캡쳐 이미지](/htm/iframe-test/img/ios_cap_2.png)

- 원본 페이지에서는 화면의 높이를 출력하였으나, iframe 내부에서는 문서의 높이값을 출력했다. 
- 테스트 결과 높이 뿐만 아니라, iframe 내부에서는 `scrollTop` 값 역시 0 이 리턴된다.
- 결국 이문제를 해결하기 위한 몇가지 방법들이 있는데, javascript 를 이용해야 해결이 가능하다.

## iframe 내부에서 중앙에 떠야 하는 레이어를 위한 꼼수
- 크로스 토메인 이슈가 없다면(두개의 도매인이 서로 같다면), javascript 를 이용해 자식, 부모간의 정보 교환이 가능하다. 즉, 자식창에서 부모창의 스크롤 값을 받아 오거나, 부모창에서 자식창의 레이어를 조정할 수 있다는 말이다. 

{% highlight html %}
    <iframe src="test_layer_js.html"frameborder="0" width="100%" height="100%" marginwidth="0" marginheight="0" id="iframeEle"></iframe>
    <script>
        $(window).on('scroll',function(){
            var layer = $('#iframeEle').contents().find('.layer_block');
            layer.css({
                'position':'absolute',
                'top' : $(window).scrollTop() + ($(window).height()/2) - (layer.height()/2)
            })
        });
    </script>
{% endhighlight %}

- 위 코드는 부모창에서 자식창에 뜰 레이어에 접근 하는 방법 이다. **테스트를 위한 코드로 절대 좋은 방법이 아님을 알린다.** scroll 이벤트 발생시마다 레이어의 위치를 갱신해주는 방법이며, 실제로 별로 좋은 방법이 아니다. 굉장히 버벅거리며, 사이트가 느려진다.
- 물론 반대의 경우도 가능하다.(자식창에서 부모창에 접근하여 갱신하는...) 하지만 어찌 되었든 버벅거리는 것은 마찬 가지이다. 그리고 매우 비 효율적이다.

- 여기서 크로스 도메인 이슈가 있다면 이야기는 완전 달라진다. 예로 부터 전해져 내려오는 고전적인 방법이 있는데, iframe 내부 페이지에 부모페이지와 같은 도메인을 사용하는 iframe 을 넣어두고 해당 iframe 을 이용해 서로 통신을 하는 방법이다. 이건 더...더...더.. 비효율적이다.

## iframe 을 none , block 시킬경우
- IOS 의 문제이다. iframe 영역을 특정 시점에서 열고 닫아야 할때 none, block 을 시키거나 iframe 을 remove 시키더라도 동일한 문제가 발생한다. iframe 영역이 사라지는 시점에서 브라우저가 스크롤 불가 상태에 빠진다.
- none 시키거나 remove 시키지 않고 position 속성을 이용해 문서 밖으로 날려 버리면 해당 증상을 해결 할 수 있다.

## 또다른 이슈...DIMD LAYER...
- 레이어가 뜨는 시점에서 레이어 뒷 배경을 투명도를 주어 약간 어둡거나 밝게 만드는 방법을 통상 적으로 dimd layer 라고 칭한다. 과거에는 modal popup 이라고도 했었다. 이 방법 역시 일반적으로 사용할 경우 별로 크리티컬한 이슈들은 없다. 단지 iframe 내부로 들어갈 경우 문제가 된다.
- 이 문제는 현제 LG 쪽에서 나온 G2, G3 의 Native App 내부 web view 에서 발생하였으며, 해당 문제를 제현하는 방법이 없어 글로 ...쓴다.
- 증상은 app 내부의 web view 영역에서 iframe 내부에 많은 양의 contents 가 작성되고 그 가운에데 dimd layer 가 오픈될 경우 .. 거기에 scroll 이 발생할 경우에 발생된다. 우선 scroll 시 배경이 깜빡 거린다.  

- 해결방법은 의외로 간단하다. css 로 해당 증상이 나타나는 때에 GPU 를 사용하도록 유도하면 된다

{% highlight css %}
.class{
  -webkit-transform: translate3D(0, 0, 0);
  -ms-transform: translate3D(0, 0, 0);
  transform: translate3D(0, 0, 0);
}
{% endhighlight %}

- dimd 되는 엘리먼트 혹은 body 에 걸거나 wrap 에 걸거나... 여튼 하면 해결 된다.

## 결론
- mobile 에서 iframe 사용은 정말 정말 피해야 한다. 특히 web 을 app 에서 webview 로 끌어와야 한다면 더더욱 피해야 한다. 이것은 개발자와 퍼블리셔, 기획자 모두를 힘들게 한다. 
- 가능한 다른 방법이 있다면 iframe 사용은 피하자. 그게 가장 현명한 방법이다.






