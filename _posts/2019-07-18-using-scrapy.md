---
layout: post
title: "Scrapy 한걸음 - 시스템 구성"
tags: [crawling, scrapy, text, python]
comments: true
---


구글에 크롤링 키워드를 검색해보자. 많은 블로그들이 여러 라이브러리로 쉽고 자세하게 크롤링 설명을 작성해둔 것을 볼 수 있다. 이 글 또한 다른 글들과 비교해보면 비슷한 내용을 포함하고 있을 것이다. 그래도 자료 정리 느낌으로 머리속 내용을 차근차근 정리하고 싶었고, 한글로 된 튜토리얼 자료가 있으면 다른 분들도 이 분야에서 삽질을 줄일 수 있지 않을까 싶어서 시작하였다. 영어로 된 docs를 보면 만사 해결이겠지만, 영어에 익숙하지 않은 본인과 같은 사람들에게 도움이 되길 바란다.  
  
---
# 기본 구조
scrapy는 파이썬으로 작성된 크롤링 프레임워크이다. BeautifulSoup를 이용해 야매로 간단하게 크롤링해본 기억만 있어서 처음에는 조금 복잡한 작동 구조를 보고 당황했었다. 

![scrapy architecture](https://docs.scrapy.org/en/latest/_images/scrapy_architecture_02.png){: .center-image}

크롤링하는데 저게 다 필요할까 생각이 들었다. 이유가 있으니까 개발자들이 이렇게 무럭무럭 키웠겠지 생각하며 일단 어떻게 구성되어 있는지 알아보자. 크게 spider, middleware, pipeline, scheduler, downloader, 그리고 engine, 이렇게 총 6개 정도로 구성되는 것을 확인 할 수 있다. 사실 자주 사용하는건 spider나 pipeline 정도밖에 없다.  

작동 흐름은 이렇게 된다.  
1. engine이 spider들을 통해 초기 크롤링 위치를 전달 받는다.  
2. engine이 scheduler에서 request 들을 관리하고, 다음 request를 요청한다. 
3. scheduler는 engine에게 다음 request를 전달한다. 

스케쥴 관리 부분이다. 빠르게 크롤링하기 위해 url들을 관리한다는 의미이다. 

4. engine은 requeust를 downloader에게 전달한다.
5. downloader는 request를 요청하여 response를 받아오고 engine에게 전달한다. 

http 요청, 응답 부분이다. 웹사이트 데이터를 가져오는 역할이다. engine과 downloader 중간에 downloader middleware 가 있는데, 이는 그 사이에서 전달되는 request나 response 전처리하기 위해서 존재한다. 

6. 의미있는 데이터를 추출하기 위해 다운받은 response를 spider에 전달한다. 
7. spider들은 필요한 부분(item)을 추출하여 engine에게 전달한다.

일반적으로 크롤링하면 생각하는 부분이다. 웹사이트에서 태그들을 해집고 다니며 의미있는 데이터를 추출하는 부분이다. 마찬가지로 engine과 spider 사이에 spider middleware가 있는데 이 또한 response나 item을 전처리하기 위해 존재한다. 

8. engine은 item을 pipeline에게 전달하고 scheduler에게 다음 request 를 요청한다. 

그리고 이 과정을 다음 request가 없을 때까지(크롤링 할 곳이 없을 때까지) 계속 반복한다. 결국 보면 빠른 크롤링을 위해 스케쥴 관리하고, 웹사이트 가져오고, 데이터 추출하고, 아이템 가공하는게 전부이다.  

# 요소 

## Engine 
engine은 크롤링 작업을 총괄하는 코어 역할을 한다. 중앙에서 거미들을 부려서 크롤러 스케쥴을 조절하고, 다운로드 계획을 세우고, 데이터 흐름을 관리해준다. 

## Spider
거미다. 웹을 돌아다니며 데이터를 가져오는 느낌의 거미다. 이게 다다. 아마 앞으로 우리가 작성할 코드에 가장 많은 요소를 차지할 것이다. 

## Pipeline 
spider들이 추출한 item을 받아서 처리하는 역할이다. 데이터베이스와 연결하여 item을 저장하거나, item들의 중복을 방지하는 그런 역할들을 할 수 있다. 

## Downloader
이름 그대로 다운로더. 웹에 request들을 요청해 response를 받아오는 역할이다. 

## Scheduler 
Request 들을 관리하여 engine에게 전달하는 역할이다. 

## Middelware 
scrapy 내부의 데이터 흐름의 중간자 역할을 한다. 데이터를 수정한다거나 중복검사를 한다거나 그런 역할들을 할 수 있다. 

지금까지 거창하게 역할들을 서술해왔지만 앞서 말했다시피 사실상 건들 부분은 spider나 pipeline 이다. 조금 더하면 middleware 이고. 너무 각 역할이 무엇인지 자세히 알려고 하지말고, 이런 흐름으로 크롤러가 동작하는구나 라는 느낌으로 보면 될 것 같다. 
더 자세한 내용은 [공식 문서](https://docs.scrapy.org/en/latest/topics/architecture.html)를 보면 된다. 
