---
layout: post
title: URL 구조 (Query Parameter, Path, etc..)
description: SEO를 고려한 URL 구조 생성 및 적절한 Query Parameter 사용하기
summary: SEO를 고려한 URL 구조 생성 및 적절한 Query Parameter 사용하기
comments: true
categories: [FE]
tags: [URL, SEO]
---

**쿼리스트링, 디렉토리?** <br>
URL 을 결정하는 기준을 어떻게 잡아야 할까 궁금해져서 적절한 URL 구조를 생성하는 방법에 대해 찾아보았다.

<br>

## 간단한 URL 구조로 생성하기

✔️ 간단하고 알기 쉬운 단어를 사용

```
https://en.wikipedia.org/wiki/Aviation
```

<br>

✔️ UTF-8 인코딩이 필요할 경우 사용 <br>
👉 [stop-using-unsafe-characters-in-urls](https://perishablepress.com/stop-using-unsafe-characters-in-urls/){:target="\_blank"}

```
https://www.example.com/%D9%86%D8%B9%D9%86%D8%A7%D8%B9/%D8%A8%D9%82%D8%A7%D9%84%D8%A9
```

<br>

✔️ 불필요하게 긴 쿼리 파라미터 피하기

```
https://www.example.com/index.php?id_sezione=360&sid=3a5ebc944f41daa6f849f730f1
```

<br>

✔️ Underscores(\_) 보다 Hyphen(-) 사용하기

```
https://www.example.com/summer_clothing/filter?color_profile=dark_grey
```

<br>

## Query Parameter 사용할 때 아래 사항 주의하기

**👉 아래 항목에 포함할 경우 문제가 생길 수도 있음을 인지할 것**
<br>

- 동적으로 생성되는 값을 파라미터 값으로 사용 <br><br>
- 세션ID 와 같은 파라미터 사용은 동일한 페이지에 대한 중복된 컨텐츠들을 생성하게 된다. <br><br>
- **sorting** 조건을 위해 너무 많은 URL 생성 <br><br>
- 내용과 관련이 없는 값을 파라미터로 넣기 ex. referrer ID <br>
  💡 위와 같은 문제를 해결하기 위해서는 `robots.txt` 을 사용해서 문제가 생길 수 있는 URL에 대한 크롤링 막기
  [robots.txt 작성 가이드](https://seo.tbwakorea.com/blog/robots-txt-complete-guide/){:target="\_blank"}

<br>
- 세션ID를 URL 에 저장하지 않고 쿠키와 같은 다른 저장소로 대체하기 <br><br>
- 필요없는 파라미터를 제외시켜서 URL 줄이기 <br><br>
- `nofollow` 같은 attribute 사용으로 동적으로 생성되는 URL 에 대한 크롤링 막기 <https://developers.google.com/search/docs/crawling-indexing/qualify-outbound-links>{:target="\_blank"}

<br><br>

## Query Parameter 역할

**Filtering, Sorting, and Pagination**

```
/products?category=electronics&sort=price_asc&page=2
```

<br>

**Tracking**: `utm` 파라미터를 사용해서 유입 경로에 대해 트래킹을 할 수 있다

```
www.yoursite.com/page?utm_source=newsletter&utm_medium=email&utm_campaign=spring_sale
```

<br>

**Search**:

```
/search?q=query
```

<br>

**Stateful Information**: 유저가 가지고 있는 상태값을 저장할때 사용. 현재 표시하고 있는 페이지 정보 또한 `Stateful information` 이 될 수 있음

<br>

**Debugging**: 디버깅의 목적으로도 사용이 가능하다

```
# 새로운 sorting 방법을 테스팅할 경우

www.yoursite.com/products?debug=enableNewSort
```

<br><br>

## Query Parameter vs Path

👉 사용되는 키워드가 페이지 전체를 identify 할 경우는 <u>Path</u> 사용 <br>
👉 페이지 안에서 filtering/sorting 의 값으로 사용될 경우에는 <u>Parameter</u> 사용
<https://stackoverflow.com/questions/30967822/when-do-i-use-path-params-vs-query-params-in-a-restful-api>{:target="\_blank"}

<br><br>

## SEO 를 고려한 URL 구조 생성하기

### search engine 이 URL을 어떻게 사용하는지 이해하기

**URL의 기본 구조**

```
protocol://hostname/path/filename?querystring#fragment
```

- `https://` 사용을 권장
- `www` 로 시작하는 도메인과 `non-www` 도메인 버전에 대해서도 서치 콘솔에 추가한다.
- `http://`, `https://` 에 대한 버전도 마찬가지로 둘 다 등록해서 관리하는 것을 권장
- protocol, hostname 은 `case-insensitive`
- path, filename, querystring 은 서버에서 어떤 페이지를 가져올지를 결정한다.
- path, filename, querystring 은 `case-sensitive` 이므로 주의할 것
- fragment 활용해 현재 브라우저에서 스크롤바가 어디에 위치하는지를 표시함. fragment 사용 없이 컨텐츠 그 자체는 같은 내용만을 인식하게 된다.
- 뒤에 붙는 슬래쉬(`/`) 표시에 대한 주의

```
동일한 내용
protocol://hostname/
protocol://hostname

다른 내용
protocol://hostname/path/
protocol://hostname/path
```

<br><br>

### 정보를 제공하는 목적의 간단한 URL로 생성하기

페이지의 내용과 관련된 URL 로 사용자에게 간단한 정보 제공하기 <br>
아래 URL 예시 참고 <br>
❌ https://www.brandonsbaseballcards.com/folder1/22447478/x2/14032015.html <br>
✅ https://www.brandonsbaseballcards.com/article/ten-rarest-baseball-cards.html <br>

<br>

### 서치결과에 URL이 들어감을 주의

**URL 에 키워드 사용하기**

피해야 하는 URL 구조 예시 <br>

- 불필요한 파라미터 사용이나 세션ID 의 사용
- page1.html 과 같은 파일명 자체를 URL에 사용
- 너무 긴 키워드 사용 ex. `baseball-cards-baseball-cards-baseballcards.html`

<br>

**깊은 구조의 계층구조를 가진 서브디렉토리 피하기**

❌ /dir1/dir2/dir3/dir4/dir5/dir6/page

<br>

**불필요한 서브디렉토리와 서브도메인명 확장**

❌ http://www.subdomain.website.com/subdirectory/subdirectory/subdirectory/page <br>
✅ http://www.website.com/subdirectory-page

<br>

**문서에 접근하는 URL을 여러개 쓰지 않기**

- `301 redirect` 로 접근해야 하는 URL 로 이동시키기
- redirect 를 사용하지 않는다면 <u>rel=“canonical”</u> link 사용하기
- ex. `domain.com/page.html` and `sub.domain.com/page.html`

<br>

**동일한 컨텐츠를 불러올때 URL이 각각 다른경우 canonical 사용하기**

- head 태그 안에 link 태그로 생성할 수 있으며 검색 결과에 보여질 URL을 선택적으로 입력할 수 있다.
- Canonical URL 에 대해 자세히 알아보기 👉 <https://www.shopify.com/partners/blog/canonical-urls>{:target="\_blank"}

<br><br><br>

**참고자료**
<br>

<https://developers.google.com/search/docs/fundamentals/seo-starter-guide?hl=en&visit_id=638185970506284552-1698876260&rd=1#hierarchy>{:target="\_blank"}<br>
<https://developers.google.com/search/docs/crawling-indexing/url-structure>{:target="\_blank"}<br>
<https://www.searchenginejournal.com/technical-seo/url-parameter-handling/>{:target="\_blank"}

<br><br>
**그 외에 읽어볼만한 문서들**<br>

[15 SEO Best Practices for Structuring URLs](https://moz.com/blog/15-seo-best-practices-for-structuring-urls){:target="\_blank"}<br>
<https://moz.com/blog/15-seo-best-practices-for-structuring-urls>{:target="\_blank"}
