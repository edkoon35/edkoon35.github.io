---
title: Gradle로 구성한 Srpingboot의 jenkins배포 방안
date: 2017-03-02 15:25:38
tags:
- gradle
- jenkins
- springboot
---

# 고민
여태껏 Maven을 이용한 빌드로 진행했던 개발을 Gradle로 바꾸는 과정에서 Jenkins 배포에 대한 문제점 발생...
Jenkins 배포 설정은 어떻게 하지?!

# 구성환경
* springboot 1.5.1.RELEASE
* gradle 3.4
* jenkins 2.6

# 해결
springboot는 jar로 빌드 후 내장 톰캣을 이용하여 단일 인스턴스만 띄울것임.(war배포는 다음에....)
다양한 jenkins 배포 설정 방법이 있을테지만 아래와 같이 심플하게 구성함.

> * Jenkins 좌측 메뉴의 `새로운 Item` 메뉴를 클릭하여 `Freestyle project`로 새로운 item 생성
![newItem](/assets/images/posts/newitem.png)

> * Build 메뉴에서 `Invoke Gradle script`를 선택 후 아래와 같이 설정함.
![Build](/assets/images/posts/2017-03-02.12.06.54.png)

> * 기본 gradle 빌드 설정후 `Add build step`에서 `Execute shell`선택
![Build](/assets/images/posts/shell.png)

> * 이번 케이스 에서는 서버에 설정되어 있는 시작/종료 스크립트를 직접 실행하게끔 구성.

끝!!
#_추가_

Spring profile을 이용한 실행은 어떻게?!!
---
검색해 보니 대부분 실행시 `"-Dspring.profiles.active=development"` 이런식으로 실행하라고 되어있는데 잘 되지 않음..ㅜㅠ
아마도 파일명으로 구분을 해서 그런거 같아요... [profile 설정관련 링크](http://jdm.kr/blog/200)
하지만 `"--spring.profiles.active=development"` 이 옵션으로 하니까 잘 됩니다!...

### springboot application.yml
```
spring:
  server:
    port: 9099
---
spring:
  profiles: local
logging:
  config: classpath:logback.xml
---
spring:
  profiles: development
logging:
  config: classpath:development/logback.xml
---
spring:
  profiles: production
logging:
  config: classpath:production/logback.xml
---
```

더 나은 방법이 있다면 알려주세요~~~~
