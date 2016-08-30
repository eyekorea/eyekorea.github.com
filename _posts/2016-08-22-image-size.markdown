---
layout: post
title:  "비율에 따른 이미지 사이즈"
date:   2016-08-15 18:00:00
categories: logic
description : "브라우저 혹은 특정 영역에 이미지를 가득 체우기 위한 방법"
---

# 이미지 가득 채우기
단순하게 생각하면 그냥 가로 100% 짜리 이미지를 브라우저에 넣으면 된다. 그런데 조금 더 생각해 보면 고려해야 할 것들이 많다.

## 조건
- 가로, 세로 각각 1000px * 600px 인 이미지가 있다. 이것을 `img` 라고 하겠다.
- document 안에 가로 100% 세로 800px 인 영역 을 갖는 element 가 있다. 이것을 `area` 라고 하겠다.
- `area` 안에는 항상 `img` 가 가득 찬 상태로 보여 져야 하며, 중앙정렬 되어야 한다.
- 하위 브라우저 지원을 위해 `transform` 속성 사용이 불가능 하다.
- 최초 접속시, 브라우저 리사이즈시에도 동일하게 동작 하여야 한다.

## 방법
- `img` 의 가로, 세로 비율값을 얻는다. 이것을 `imgRatio` 라고 하겠다.
- `area` 의 가로, 세로 비율값을 얻는다. 이것을 `areaRatio` 라고 하겠다.

## 공식
- `areaRatio` 를 기준으로 각각의 비율에 따라 아래와 같은 함수를 만들어 볼 수 있다.
- 궂이 비율을 계산하여 (1:1.4 와 같은) 비교 한 이유는, 가로, 세로 실제 값으로 비교하게 되면 그에따른 경우의 수가 더 많아져 복잡해 지기 때문이다. 예를 들어 어느 한쪽은 반듯이 1이 되는 비율로 계산해 놓으면 차후에 둘중 값이 큰값을 찾아 서로 비교하면 그 이후의 값에 대해서 더이상 비교할 필요가 없기 때문...?

{% highlight js %}
  function ratio ( w, h ) {
      return ( w > h ) ? [w/h, 1] : [1,h/w];
  }

  function imgSizeFnc ( imgWidth, imgHeight , areaWidth, areaHeight ){
    var imgW = imgWidth,
      imgH = imgHeight,
      blockW = areaWidth,
      blockH = areaHeight,
      imgRatio = ratio ( imgW, imgH ),
      blockRatio = ratio ( blockW, blockH );
    var _class = (function(){
      if( blockRatio[0] > blockRatio[1] ) { // 화면 가로 비율이 더 크다.
        if( imgRatio[0] > imgRatio[1] ) { // 이미지도 가로 비율이 더 크다.
          if ( blockRatio[0] > imgRatio[0] ) {
            return 'imgW';
          } else {
            return 'imgH';
          }
        } else { // 이미지는 세로 비율이 더 크다
          return 'imgW';
        }
      } else { // 화면 세로 비율이 더 크다.
        if ( imgRatio[0] < imgRatio[1] ){ // 이미지도 세로 비율이 더 크다.
          if( blockRatio[1] > imgRatio[1] ) {
            return 'imgH';
          } else {
            return 'imgW';
          }
        } else {
          return 'imgH'
        }
      }
    })();
    return _class;
  };
{% endhighlight %}

- `ratio` 라는 함수는 두개의 인자( `:Number` )를 받는다. 그리고 두개의 인자를 비교하여 비율을 리턴한다.
- `imgSizeFnc` 함수는 네개의 인자( `:Number` )를 받는다. 각각 `img` 의 가로, 세로 그리고 `area` 의 가로세로이다. 이 함수에서는 `ratio` 에서 리턴 받은 값을 기준으로 비교하여 단순히 `img` 에 가로값을 대입할지 세로값을 대입할지에 대한 문자열( `:String` )만 리턴한다. 
- 이렇게 리턴받은 문자열에 따라 `area` 의 가로값을 대입하고 세로값을 auto 로 지정하거나 혹은 `area` 의 세로값을 대입하고 가로값을 `auto` 로 지정하는 방식으로 `img`를 가득 채울 수 있다. 단, 여기서 고려해야 할 사항이 있다. 위의 조건에서 이미지는 항상 중앙에 위치 해야 한다. 이에 따라 style 이 반영된 상태에서 다시 left 값이나 top 값의 조정이 이루어 져야 한다.

- 뭐.... 요부분은 크게 어려운 부분은 아니니 넘어가는 것으로 하면서...급하게...

# 마치며,
일단 뭐 공식 자체는 간단하지만 리사이즈 될때마다 여러장의 이미지들이 계속해서 저런식으로 갱신되는 것은 바람직 하지 않다.  
따라서 현재 화면에 보이는 것들 혹은 화면에 보이기 직전의 것들에 대해서만 갱신 하도록 분기 처리를 해두는 것도 나쁘지 않은 방법일듯 하다.  