---
layout: post
title: github blog 생성해보기
description: github blog를 생성해보는 과정을 적은 포스트입니다.
summary: github blog를 생성해보는 과정을 적은 포스트입니다.
comments: true
categories: [gitblog]
tags: [jekyll, gitblog, ruby, bundle]
---

### github blog 생성

개발블로그 시작을 그렇게도 망설이고 미뤘는데 이왕이면 블로그 생성, 운영 하면서 겪는 경험도
도움이 많이 될 것 같아 얼레벌레 시작하게 된 블로그.<br>
욕심 부리지 않고 천천히 배워볼 예정이다. 👏<br><br>

- 레파지토리 생성<br>
  `github-pages` 배포를 위해 반드시 아래 규칙을 따라 레파지토리를 생성해야 한다.

```
username.github.io
```

- 맘에드는 `jekyll theme` 을 적용하기<br>
  무료테마를 고른 후, `git clone` 하여 일단 테마의 모든 내용을 내 깃헙 블로그 레파지토리 디렉토리로 옮기는 것으로 시작한다. 다 옮겼다면 커밋, 푸시를 한다.

- 1~3분 정도 지나면 커밋된 레파지토리 변경 내용이 `github-pages` 를 통해 배포가 되어있고, `username.github.io` 로 들어가서 배포된 사항을 확인할 수 있다.

- 로컬에서 블로그를 실행하려면 다음과 같은 과정이 필요하다고 한다.<br>
  `jekyll` 과 `ruby` 설치가 필요한데, mac OS 환경의 설치 방법은 다음 doc을 참고하였다.<br>
  <https://jekyllrb-ko.github.io/docs/installation/macos/>

<br>

### 로컬설치 참고

`rbenv` 를 통해 `ruby` 를 설치하면 프로젝트마다 여러 버전을 설치할 수 있다고 하는데<br>
성격 급한 나머지 이 내용을 못보고 `ruby`를 설치해버린 나..<br>
근데 `ruby` 를 평소에 배워본적도 사용해본적도 없는 상태이므로 일단은 나중에 필요하면 설치하도록 하고 다음단계로 넘어갔다.<br><br>

로컬에서 실행해보려고 했는데 여기서 몇 가지 문제점들이 발생했다.<br><br>

1. bundler, bundle은 무엇인가? 😂 npm 같은 존재인가..
2. Gemfile 에 적용된 gem들의 버전이 다 deprecated 된 상태라 도통 실행이 되질 않는다.
3. 2번의 문제를 해결하면 실행은 할 수 있을 것 같은데 오늘 처음 본 bundle, gem.... 하아...^^

<br>
해결 방법
<br>

1. `bundle update --bundler` 를 통해 bundler 버전을 업데이트 해주었다.<br>Gemfile에 적용된 버전은 지원하지 않는 버전인데 무슨 버전으로 업데이트 해야할지 몰라서 latest 로 설치했다.<br><br>
2. `bundle outdated` 하면 현재 쓰는 gem 중에서 업데이트 대상에 속하는 gem 들을 알려주는데 뭐가 뭔지도 모르겠고 거의 모든 gem이 outdated 상태여서 모든 gem을 다 업데이트 해주기로 했다.<br><br>
3. `bundle update --all` 로 outdated 된 gem을 모두 업데이트 해준다.<br><br>
4. 1~3번까지 모두 했는데도 또 오류가 난다. ^^<br><br>
5. 오류 로그를 보니 webrick 파일을 로드할 수 없다는 내용이었다. 음....<br><br>
   <img width="720" alt="webbrick-cannot-load-error" src="https://user-images.githubusercontent.com/56527636/182014515-7acd4add-9b23-4760-ae63-6a2f49ca22b4.png"> <br><br>
6. 구글링을 해보면 `webbrick` 또한 설치를 해주면 된다고 하니, `bundle add webrick` 로 설치를 진행한다.<br>
   🔗 webrick이란? <https://github.com/ruby/webrick> <br><br>
7. `bundle exec jekyll serve` 로 서버를 실행한 후 `localhost:4000` 으로 접속하면 로컬 실행이 마무리 된다.<br>
   <img width="720" alt="jekyll-serve" src="https://user-images.githubusercontent.com/56527636/182014485-6e044f84-3e46-4832-ace4-7cd1565b8955.png">

<br><br>

이제 하나하나 커스텀하는 과정은 아래 리소스를 참고해서 진행할 예정이다.<br>
<https://jekyllrb-ko.github.io> <br>
<https://bundler.io>
