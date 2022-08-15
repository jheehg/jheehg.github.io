---
layout: post
title: github actions으로 jekyll 수동 배포
description: github actions으로 jekyll 수동 배포를 위한 글입니다.
summary: github actions으로 jekyll 수동 배포를 위한 글입니다.
comments: true
categories: [gitblog]
tags: [jekyll, github actions]

---
<br>

### github actions로 배포하게 된 이유
---
jekyll 에서는 카테고리나 태그 같은 기능을 사용자가 직접 플러그인을 생성해서 사용할 수 있게 하는데 <br>
`github-pages` 로 자동 배포하게 될 경우 커스텀 플러그인을 사용할 수 없게 된다.<br>

아래 jekyll 공식 docs에서 github actions과 plugin에 관련된 설명이다.
```
When building a Jekyll site with GitHub Pages, the standard flow is restricted for security
reasons and to make it simpler to get a site setup. For more control over the build and 
still host the site with GitHub Pages you can use GitHub Actions.

Plugins — You can use any Jekyll plugins irrespective of them being on the supported
versions list, even *.rb files placed in the _plugins directory of your site.
```
<br>
이로 인해 `category` 플러그인 생성 후 실제 배포를 하면 `/tag/*`, `/category/*` 가
`404 not found` 페이지로 이동했다. <br>
(아래와 같이 `_plugins`  에서 tags는 가져왔던 테마에 원래 있었고 category는 새로 생성한 상태였다.)

<img width="300" alt="jekyll_plugins" src="https://user-images.githubusercontent.com/56527636/184523048-6270d2f0-7fa6-4a9d-b4ca-1ea062f06c8f.png">


구글링을 통해 현재 상태에서 해결할 수 있는 방법을 찾았으나 실패했다.
<s>master 브랜치에 .nojekyll 파일을 _site 폴더와 같이 커밋해서 배포하는 방법..?</s>

<br><br>

### github actions에서 새 workflow 생성
---
workflow 파일의 경우 github actions 에서 제공하는 것도 있고 marketplace에서 찾을 수도 있는데<br>
나는 아래 github에 예시와 jekyll을 수동 배포한 다른 블로그 글들을 보고 참고해서 작성했다.<br>
<https://github.com/helaili/jekyll-action>

<br>

### workflow 배포하면서 실패했던 원인들을 정리해보기

```yml
name: Build and deploy Jekyll site to GitHub Pages

on:
  push:
    branches:
      - master

jobs:
  github-pages:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/cache@v2
        with:
          path: vendor/bundle
          key: {% raw %}${{ runner.os }}-gems-${{ hashFiles('**/Gemfile') }}{% endraw %}
          restore-keys: |
            {% raw %}${{ runner.os }}-gems-{% endraw %}
      - uses: helaili/jekyll-action@2.0.5    # Choose any one of the Jekyll Actions
        with:
          target_branch: 'gh-pages'
        env:                                # Some relative inputs of your action
          JEKYLL_PAT: {% raw %}${{ secrets.JEKYLL_PAT }}{% endraw %}
```

<br>

#### 🔗 target_branch 를 `gh_pages` 로 지정해주어야 `_site` 디렉토리 안에 정적파일들이 해당 브런치에 생성된다.

#### 🔗 workflow에 secrtes 지정할 때 key, value값 주의하기
PAT 추가해주는 부분에 key값을 어떻게 입력해야 될지 확실하지 않아서 이 부분에서 여러번 배포를 실패했다. <br>
PAT 키 생성과 input입력은 아래 과정과 같이 해결했다. <br><br>

- 레파지토리에서 settings 탭으로 이동한다.<br><br>
<img width="800" alt="repo_settings" src="https://user-images.githubusercontent.com/56527636/184523331-a61d8264-1077-4ebc-8e21-0546c6102fb4.png"><br><br><br>

- Security 탭에서 Action 탭으로 이동한다.<br><br>
<img width="280" alt="security_page" src="https://user-images.githubusercontent.com/56527636/184523358-986607ad-31e0-4a2e-9c9f-62421d5276ab.png"><br><br><br>
	
- Repository secrets에 토큰 값을 추가해준다. <br>이 토큰 값은 github 개인 설정 페이지에서 Developer settings - Personal access tokens 에서 생성 해준 값을 사용한다.<br><br>
<img width="650" alt="repo_secrets" src="https://user-images.githubusercontent.com/56527636/184523439-19ee3b91-c157-475b-a4b3-ded07d11d55a.png"><br><br><br>

- workflow file에 Repository secrets input으로 key값과 value값은 각각 <br> `JEKYLL_PAT`, `{% raw %}${{ secrets.JEKYLL_PAT }}{% endraw %}` 으로 넣어준다. secrets 명은 임의로 생성한다.<br>

아래는 github actions workflow security guide 관련 문서인데 secrets 생성하는 규칙이 나와있다. <br>
<https://docs.github.com/en/actions/security-guides/encrypted-secrets> <br><br><br><br>

#### 🔗 배포 소스 파일의 브랜치 설정을 `gh-pages` 로 지정하기
- 레파지토리의 settings 탭에서 `Pages` 페이지로 이동한다.<br><br>
<img width="280" alt="pages" src="https://user-images.githubusercontent.com/56527636/184597903-ff99a5f1-3fea-4329-ace1-b87f86fba142.png"><br><br><br>

- Build and deployment 에서 branch 지정을 `master`에서 `gh-pages`로 변경해준다.<br><br>
<img width="700" alt="build-branch" src="https://user-images.githubusercontent.com/56527636/184597964-1aa61cd3-434c-464f-83ed-d247b4e595fa.png"><br>

아래 블로그 글에 해당 내용이 잘 나와 있다.<br>
<https://bitbra.in/2021/10/03/host-your-own-blog-for-free-with-custom-domain.html>
<br><br>

### 추가로 더 알아봐야 할 내용들
`github-pages`에서 자동배포 되는 action이 수동으로 생성한 action과 중복해서 실행되고 있는데 <br>
수동으로 생성한 workflow만 실행되게 할 수 있을지 알아보려고 한다. 

<br><br><br>

#### github actions 참고 자료
<https://docs.github.com/en/actions><br>
<https://jekyllrb.com/docs/continuous-integration/github-actions/#advantages-of-using-actions>