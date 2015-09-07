---
layout: post
title:  "yo error eacces permission denied"
date:   2015-09-07 00:00:00
categories: yeoman
description : "yeoman error eacces permission denied"
---

##yeoman 권한 에러

```
$npm install -g yo
$yo

Error: EACCES, permission denied '/Users/userid/.config/configstore/insight-yo.json'
You don't have access to this file.

    at Error (native)
    at Object.fs.openSync (fs.js:500:18)
    at Object.fs.readFileSync (fs.js:352:15)
    at Object.create.all.get (/usr/local/lib/node_modules/yo/node_modules/configstore/index.js:27:26)
    at Object.Configstore (/usr/local/lib/node_modules/yo/node_modules/configstore/index.js:20:44)
    at new Insight (/usr/local/lib/node_modules/yo/node_modules/insight/lib/index.js:37:34)
    at Object.<anonymous> (/usr/local/lib/node_modules/yo/lib/cli.js:128:15)
    at Module._compile (module.js:460:26)
    at Object.Module._extensions..js (module.js:478:10)
    at Module.load (module.js:355:32)

```  

angularJS 를 공부 하기 위해 `yeoman` 을 사용하던 중 개인 맥북이 아닌 회사 아이맥에서 위와 같은 권한 오류가 났다.  
본인 역시 맥을 사용하기 시작한게 불과 1년 남짓이라 열심히 구글링을 한 끝에 권한에러 해결 방법을 찾아냈다.  
해결 방법은 [이곳](http://www.jaychen.me/book/angularjs/issues/error-eacces-permission-denied-rootconfigconfigstoreinsight-yojson)에서 확인 했으며, 본인은 다행히도 성공했다.  
위와 같은 에러가 났다면, `userid` 항목에는 본인의 맥 계정이 들어가 있을 것이다. 위의 블로그에서 제안한 해결 방법은 두가지이다.

###사용하고 있는 노드모듈을 업데이트 한다.

```
$ npm install -g npm stable
```

###문제가 있는 디렉토리에 권한을 변경한다.
```
$ chmod g+rwx /root /root/.config /root/.config/configstore
```

**위의 `root` 는 개인 설정에 따라 다를 수 있으나 통상 `/Users/userid/` 이며, userid 를 모르겠다면 아래와 같이 확인 할 수 있다.**

```
$whoami
```
