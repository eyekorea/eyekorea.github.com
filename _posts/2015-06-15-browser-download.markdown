---
layout: post
title:  "모바일 브라우저의 정적구성요소 다운로드 테스트"
date:   2015-06-15 00:00:00
categories: mobile-test
description : "모바일 브라우저의 정적 구성요소 다운로드와 javascript blocking"
---

##호기심에서 시작.
- 모바일 브라우저는 몇개의 이미지와 css 를 동시에 다운로드 할까?
- 중간에 js 파일이 들어가 있다면 과연 [javascript blocking](https://developers.google.com/speed/docs/insights/BlockingJS) 이 정말 발생할까? 그것이 미치는 영향은 어느정도 일까?

##Test 방법
1. 500Kb 용량의 이미지를 10개 준비.
1. html 파일에서 해당 이미지를 로드하도록 마크업.
1. mobile 브라우저에서 개발자 도구를 열어 network 항목을 체크
1. 상, 중, 하단에 js 파일을 삽입하여 network 항목을 체크

##Test 예상
- 몇개씩 동시 다운로드 할지에 대해 전혀 예상이 안됨.
- js blocking 으로 인한 속도 감소는 미비하지 않을까?

##Download 갯수
- 모바일 크롬 브라우저와 IOS Safari 브라우저는 모두 한번에 6개의 이미지를 다운로드 받는다.
- 최초 6개의 다운로드가 한번에 이루어 지며, 다운로드 종료가 되는 시점에서 다음 이미지를 다운로드 한다.(6개 모두 다운로드 완료할때까지 대기하지 않는다.)
아래와 같이 6개씩 동시 다운로드가 이루어 지는 것을 확인 할 수 있었다. 이제 중간에 `script block` 이 들어갈 경우 미치는 영향이 어느정도인지 궁금해 졌다.

<img src="/assets/images/postimg/20150615/mobile_chrome_download_len.png">
*mobile chrome browser*  

<img src="/assets/images/postimg/20150615/mobile_safari_download_len.png">
*mobile safari browser, 처음 6개를 받은 후 종료 시점에 따라 다음 이미지를 다운 받는다.*

- - -

##inline Script block
아래와 같은 코드를 사용했다.  
{% highlight html %}
    <img src="TEST1.jpg" alt="" width="100%">
    <!-- ##### 생략 ##### -->
    <img src="TEST8.jpg" alt="" width="100%">
    <script>
        var i=0,
            tag = document.createElement('p'),
            arr = [];
        for(;i<19000000;){
            arr.push(i);
            i++;
        }
        tag.appendChild(document.createTextNode(i));
        document.body.insertBefore(tag,document.getElementById('testimg'));
    </script>
    <img src="TEST9.jpg" alt="" width="100%">
    <!-- ##### 생략 ##### -->
    <img src="TEST19.jpg" alt="" width="100%">
{% endhighlight %}
  
일단, 중요한 것은 영향이 있는가 없는가 였다.  
궁금한 것은 문서 중간에 스크립트 블럭이 있을때, 순차적으로 다운로드를 받다가 스크립트 블럭을 만났을때, blocking 이 발생하는지에 대해서 였기때문에 브라우저에 부하를 주기위해(블럭킹 현상을 보고 싶었다.) 모바일 크롬이 견딜 만큼의 루프를 돌려서 배열에 push 했다.  
  
  
사실 js blocking 은 너무 널리 알려진 이슈이기 때문에 순조롭게 테스트 하고 포스팅 하면 될것이라 생각했다. 하지만 몇번을 테스트 해도 만족할만한 값이 나오지 않았다.  
마치 *원하는 결과를 얻기 위해 미련한 테스트를 계속하는 것* 같았다. 순수하게 결과를 받아 들이고 있는 그대로를 포스팅 해야 하는데, 내가 알고있는 것과 결과가 달라 조금 당황했다.  
아래는 그 결과 이다.
- - -
<img src="/assets/images/postimg/20150615/m_chrome_download_1.png">
<img src="/assets/images/postimg/20150615/m_chrome_download_2.png">
<img src="/assets/images/postimg/20150615/m_chrome_download_3.png">
위 이미지 세개는 자바스크립트가 들어가 있지 않은 상태에서의 Chrome Network 그래프를 캡쳐한 화면이다. 문서를 먼저 받고 6개씩 이미지를 로드한다.
  
  
- - -
<img src="/assets/images/postimg/20150615/m_chrome_blocking_1.png">
<img src="/assets/images/postimg/20150615/m_chrome_blocking_2.png">
<img src="/assets/images/postimg/20150615/m_chrome_blocking_3.png">
위 이미지 세개는 자바스크립트가 들어간 상태에서의 Chrome Network 그래프를 캡쳐한 화면이다. 자바스크립트가 삽입되지 않은 상태에서의 문서와 별다른 모습이 보이지 않고있다.

- - -
포스팅에 자세히 적지는 않았지만 이 그래프를 좀 눈에 띄게 다른 그래프로 만들고 싶어 위의 자바스크립트 블럭에 별것을 다 해봤다. 하지만 Network 크래프는 별다른 차이가 없었다.  
계속 보다보니, 그래프가 아닌 디바이스의 화면이 그려지는 속도에 차이가 있음을 감지했다. 이에 Network 가 아닌 Timeline 을 참조하여 다시 테스트를 진행했으며, 결과는 아래와 같다.  
  

- - -
<img src="/assets/images/postimg/20150615/m_chrome_download_timeline.png">
자바스크립트가 들어가 있지 않은 문서의 Timeline 이다.

<img src="/assets/images/postimg/20150615/m_chrome_blocking_timeline.png">
자바스크립트가 들어간 문서의 Timeline 이다.
  

아래 이미지의 *Evaluate Script* 부분이 위에서 브라우저에 부하를 주기위해 강제로 삽입한 스크립트를 처리하기 위해 걸린 시간을 나타낸다.  
자바스크립트는 정확히 8번째 이미지 이후에 있음에도 불구하고, 해당 스크립트를 처리하기까지 무려 4초간 화면에 그 어떤것도 표시하지 않았다. 앞서 봤던 Network 그래프를 보면 알듯이 이미지의 download 는 스크립트가 있던, 없던 비슷하게 진행했으나 paint 가 발생하는 시점이 확실한 차이를 보여 주었다.  
개인적으로 굉장히 놀라운 결과였던 것은, html Parser 가 dom node 를 구성한뒤 순서대로 request 를 발생하면서 paint 를 진행하다, javascript block 을 만났을때 download 를 멈추고 스크립트를 처리한뒤 paint 를 다시 진행할 줄 알았으나, paint 조차 진행하지 않고 download 만 받아두고 스크립트를 처리한 뒤에서야 paint처리하여 화면에 뿌려준 것이다.  
그럼 Mobile 이라 그런걸까..?  
아래는 크롬 PC 브라우저의 Timeline 이다.

- - -
<img src="/assets/images/postimg/20150615/PC_chrome_blocking_timeline.png">

놀랍게도 PC 역시 동일했다. 사용자의 체감속도(화면이 뜨는)를 빠르게 하기 위해서 가장 중요한 것은 Paint 되는 시점...그러니까 실제로 화면이 뿅 하고 뜨는 시점인 것이다. 그런데 내가생각 하던 시점과는 조금 다른 결과가 나와 조금 놀랐다.

그럼 iOS safari 에서는 어떻게 처리 하고 있을까..?
아래는 그 결과이다.  
- - -
먼저 safari 개발자 툴이 자꾸 비정상 종료 되어 roof 수를 190000 으로 줄이고 테스트를 진행하였다.
<img src="/assets/images/postimg/20150615/m_safari_download_dbg.png">
사파리 브라우저 역시 자바스크립트 블럭이 없을경우 6개씩 원할한 다운로드 및 paint 를 진행한다.
  
  
<img src="/assets/images/postimg/20150615/m_safari_blocking_dbg.png">
다만 자바스크립트가 중간에 삽입되었을때, 크롬브라우저와 조금 다른 처리 방법을 볼수 있었는데, js block을 만나는 순간 처리가 끝날때까지 다음 이미지를 다운로드 받지 않았다. 물론 js block 을 만나기 전까지 paint 를 진행하지 않는것은 동일했는데, js 를 처리하는 시간을 길게 하고 테스트 하고 싶었으나, 개발자 도구가 자꾸 멈춰서 원만한 테스트가 어려웠다.

- - -
여기까지의 테스트를 통해 우선 javascript 가 문서 중단에 있을경우 paint 를 전혀 진행하지 않고, safari 는 download 까지 중단 한다는 것을 알 수 있었다. 다만 의문은 문서 하단에 위치했을때...  
그래서 이번에는 문서 하단으로 스크립트 블럭을 옮겨 보았다.
<img src="/assets/images/postimg/20150615/m_chrome_blocking_timeline_bottomjs.png">
음...? 결과가 동일하다. 스크립트에 문제가 있는 것일까..? 스크립트 코드를 수정해 보았다.  

우선 전역에서 실행되는 스크립트 코드를 함수로 변경후 실행시키지 않았다. 결과는 아래와 같다.
<img src="/assets/images/postimg/20150615/m_chrome_blociking_fnc.png">
분석해 보자면, 바로 실행을 시켰을때 약 4000ms 정도 blocking 되던 부분이 1.546ms 정도로 줄었다. 거의 티도 안난다고 봐야 겠다.  
함수를 바로 실행시키면 어떨까..? 아쉽게도 결과는 동일했다. 기존과 동일하게 3000~4000ms 정도로 blocking 이 발생했고, 그동안 브라우저는 paint 하지 않았다.  
그럼 document ready 되었을때는 어떨까...? 아... 일단 테스트 특성상 jquery 자체를 load 하지 않았는데, 고민에 빠졌다. jquery 를 추가로 로드할까...? 아니면...만들까...하다가...  
여기서 jquery 가 상단, 하단 에 각각 삽입되었을때의 테스트를 함께 진행하면서....다시 정리를 해보기로 했다.

- - -
##jquery load
우선 상단에서 로드 해 보았다.

{% highlight html %}
<head>
    <!-- #### 코드 생략 #### -->
    <script src="//code.jquery.com/jquery-1.11.3.min.js"></script>
</head>
<body>
{% endhighlight %}
<img src="/assets/images/postimg/20150615/m_chrome_blocking_insertjQuery_top.png">

jquery 로 인해 122ms 정도 blocking 이 발생했고, 추가로 function call 이 8.210ms 정도 발생했다. 재미있는건 Network 쪽이었는데, js 파일을 불러오면서 동시에 이미지를 다운로드 하긴 하는데, 1장만 함께 다운로드 했다.
<img src="/assets/images/postimg/20150615/m_chrome_blocking_insertjQuery_top__network.png">
즉, js 파일 하나가 위에 있으니 6장씩 다운로드를 진행하던 것이 2개로 바뀐것이다. 여기에 js 파일을 하나 더 추가하면 어떨까?
<img src="/assets/images/postimg/20150615/m_chrome_blocking_insertjQuery_two_js.png">
js 두장을 함께 받으며, 이미지 한장을 함께 다운로드 받았다. 여기서 의문이 생겼다...그럼 js 파일은 몇개까지 동시에 다운로드 할까..? 이부분은 다음번에 디테일하게 포스팅 하도록 하고, 문서 하단으로 js 로드를 옮겨보기로 했다.
<img src="/assets/images/postimg/20150615/m_chrome_blocking_insertjQuery_bottom_network.png">
js파일의 로드를 하단으로 이동하니 이미지 6장을 먼저 다운로드 한다. 이것은 굉장히 중요한 부분인데, js를 로드할때 상단에 있을때보다 하단에 있을때 좀더 효과적으로 다운로드를 진행 할 수 있다는 의미이다. 우리는 보통 마크업을 진행할때 상단에서 부터 순서대로 마크업하고, 이때 삽입되는 이미지들은 상단 부터 순서대로 로드 하게 된다(테스트상 처음 6개의 이미지는 순서대로 받아진다).  
사용자에게 보여질 필요한 이미지를 먼저 로드해서 화면에 노출하여 준다는 것은 체감상 굉장한 효과를 주게된다. 다만 재미있는 것은 문서 최 하단에 두었음에도, 6개의 이미지를 로드한 이후에 js 파일을 먼저 로드하는 것인데, 그럼, js 파일을 이해할때까지 브라우저는 paint 를 하지 않을까?

<img src="/assets/images/postimg/20150615/m_chrome_blocking_insertjQuery_bottom_timeline.png">
문서 하단에 js 파일을 위치 하게 되니, js 파일을 로드하고 이해하기 전에 이미 다운받은 이미지에 대한 paint 를 시작했다. 화면에 노출시키고 있다는 것이다. 이로써 js 파일이 상단에 있을때와 하단에 있을때의 차이와 효과는 분명해 졌다.(ySlow 와 pageSpeed 항목이 거짓은 아닌 것이다.)  
이제 document.ready 테스트를 진행해 보기로 했다.

##document Ready
테스트는 간단하다. 그냥 아래 코드에서 함수를 실행하면 되는데..

{% highlight javascript %}
$(function(){
    testFnc();
})
{% endhighlight %}
<img src="/assets/images/postimg/20150615/m_chrome_blocking_ready_fnc.png">

처음 전역에서 바로 함수를 실행했을때와는 달리 paint를 진행하는 중간에 함수를 실행한다.  
차이는 스크립트가 실행되는 동안 paint 를 시작조차 하지 않아 멈춰 있던 화면이 이제는 화면을 그리는 중간에 함수를 실행해서 사용자 체감 속도가 증가한 샘이다.  

##정리
- 모바일 브라우저역시 6개씩 정적 구성요소를 다운로드 했다.
- iOS 의 safari 는 inline script block 을 만났을때 정적구성요소를 다운로드 하지 않고 대기한다.
- android 의 chrome 은 inline script block 을 만났을때 정적구성요소를 다운로드 받지만 paint 하지 않는다.
- inline script block 이 실행될때 브라우저는 이전에 로드했던 이미지조차 paint 하지 않고 스크립트를 먼저 실행한다.
- js 파일이 head 에서 src 로 로드할때 브라우저는 1개의 정적 구성요소를 추가로 다운로드 받는다.
- js 파일이 문서 하단에 있을때, 브라우저는 상위에 있는 6개의 정적 구성요소를 로드한뒤 하단의 스크립트 파일을 먼저 로드한다.
- 문서 하단에서 바로 실행하는 것 보다 문서가 준비되고 load 및 paint 가 진행되는 시점(document.ready)에 함수를 실행하는 것이 더 효과 적이다.

##차후 예정 테스트
- 브라우저는 js 파일을 동시에 몇개까지 로드할까?
- css 파일을 로드하는동안 css 내에서 사용되는 background 이미지와 html 문서상의 이미지중 어떤것을 먼저 로드할까?

##마치며
이미 위 내용들중 큰 틀은 ySlow 나 pageSpeed 의 항목에 모두 포함되어 있다. 단지 css는 상단에, js 는 하단에...와 같이 답만 나와 있을뿐, 어떻게 처리하고 어떤 일이 발생해서 그렇게 해야 하는지. 반듯이 모든 상황에서 그것이 맞는지 의문이 들어서 테스트를 진행했다.  
이 포스팅을 위해 대략 2~3일 정도 삽질을 무한 반복 한듯 하다.  
이제 이것을 어떻게 받아들일지는 이 글을 읽어준 당신의 몫이다.

포스팅을 위해 만든 테스트 파일은 [이곳](https://github.com/eyekorea/browser.download.test.git)에서 받아 볼 수 있다.