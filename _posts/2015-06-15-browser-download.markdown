---
layout: post
title:  "모바일 브라우저의 동시 다운로드"
date:   2015-06-15 00:00:00
categories: mobile-test
---

#Mobile browser의 동시 download
- 모바일 브라우저는 몇개의 이미지와 css 를 동시에 다운로드 할까?
- 중간에 js 파일이 들어가 있다면 과연 [javascript blocking](https://developers.google.com/speed/docs/insights/BlockingJS) 이 정말 발생할까? 그것이 미치는 영향은 어느정도 일까?

##Test 방법
1. 1.29Mb 용량의 이미지를 38개 준비.
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
*mobile safari browser*

- - -

##inline Script block
아래와 같은 코드를 사용했다.  
{% highlight html %}
    <img src="TEST1.jpg" alt="" width="100%">
    <!-- ##### 생략 ##### -->
    <img src="TEST8.jpg" alt="" width="100%">
    <script>
        var testConsole = 'blocking Script Test',
            obj = document.getElementById('testimg'),
            tag = document.createElement('p'),
            i=0,j=0,nextTxt;
        for(;i<10000;){
            for(;j<100;){
                j++;
            }
            j=0;
            i++;
        }
        nextTxt = document.createTextNode(testConsole+(i+j));
        tag.appendChild(nextTxt);
        document.body.insertBefore(tag,obj);
    </script>
    <img src="TEST9.jpg" alt="" width="100%">
    <!-- ##### 생략 ##### -->
    <img src="TEST19.jpg" alt="" width="100%">
{% endhighlight %}
  
일단, 중요한 것은 영향이 있는가 없는가 였다. 
궁금한 것은 문서 중간에 스크립트 블럭이 있을때, 순차적으로 다운로드를 받다가 스크립트 블럭을 만났을때, blocking 이 발생하는지에 대해서 였고 정확한 시점을 보고싶어서 alert 을 이용했다.


