---
layout: post
title:  "scrollTop 값에 따른 엘리먼트의 값"
date:   2016-08-15 18:00:00
categories: logic
description : "javascript 개발시 scrollTop 값의 변화에 따른 element 위치값 체크"
---

#시작하며
로직을 조금씩 기록해 두어 향후 있을 개발에 불필요한 시간을 사용하지 않는데 목적이 있다.  
엘리먼트가 화면에 표시되는 순간부터 사라질때 까지를 확인 할 수 있는 방법이다.

#상황
![설명 이미지](/assets/images/postimg/20160816/img_1.png)

- 위 이미지의 파란색 화면 부분은 윈도우즈 브라우저이다. 그리고 하단의 분홍색은 스크롤 해야지 볼수 있는 컨텐츠 영역이다.
- 분홍색 컨텐츠가 화면 하단에 나타나기 시작하면서 부터 화면위로 사라질때까지, 혹은 그 반대일 경우를 확인한다.

#구현.
{% highlight js %}

function activeView () {
  var $ele = $( "#viewElement" ), // 체크할 엘리먼트
       $win = $( window ), // 윈도우
      $output = $( "#activePercent" ); // 동작 확인을 위한 출력 값 노출 영역(input)

  var checker = function(){
    var sPoint = $ele.data( "offsetTop" ) - $win.height(), // 시작값.
        ePoint = $ele.data( "offsetBottom" ), // 종료값.
        // 해당 영역이 화면에 들어와서 나가기까지 몇퍼센트 진행되었는지 값을 산출.
        percent = Math.abs( 
          ( ePoint - $win.scrollTop() ) / ( ePoint - sPoint ) * 100 
        );
     if ( sPoint < $win.scrollTop() && ePoint > $win.scrollTop() ) {
       $output.val( percent ); // 화면안에 있음.
     } else {
       $output.val( "out" ); // 화면인에 없음.
     }
  };

  // 엘리먼트에 위치값을 최초에 저장.
  $ele.data({
    "offsetTop" : $( $ele ).offset().top,
    "offsetBottom" : $( $ele ).offset().top + $( $ele ).height()
  });

  // 윈도우에 스크롤 이벤트 추가.
  $win.on( "scroll", checker );
}

// 이미지 로드 등의 이슈로 인해 최초 offsetTop 값을 정확히 저장하지 못할 수 있으므로,
// load 된 이후에 함수 실행.
$( window ).on( "load", function(){
  activeView();
} );


{% endhighlight %}

- 영역이 안으로 들어왔는지 확인하기 위해, 퍼센트를 구해 input 필드에 노출 시켰다.
- [이곳](http://jsbin.com/bomifus/edit?html,css,js,output) 에서 확인 할 수 있다.

##마치며
항상 만들어 놓고 나면 간단한 공식인데, 막상 시작할때 고민을 많이 하게 된다.  
같은 로직을 매번 고민하지 않기를...