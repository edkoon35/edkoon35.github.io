---
title: Logstash를 이용한 실시간 서비스 데이터 동기화 방안
date: 2017-08-01 15:06:47
tags: logstash
---

내부 서비스의 데이터를 외부 서비스로 동기화해야 하는 경우가 생겼다.
거의 모든 데이터를 줘야하는 상황으로 발전할 수도 있을것 같아 정말 맘에 안들긴 하지만 그건 그때 생각해 보기로 하고(~~정말 그런 수준이라면 실시간으로 서비스 API를 호출하는게 맞다.!!~~)
이기종간의 데이터 동기화에는 다양한 방법이 있겠지만 이번에 처리한 내용을 정리한다.

* 데이터 변경이 이뤄질때마다 실시간으로 동기화
* 변경이 이뤄진 시점과 대상은 우리만 알고 있다
* 데이터의 양이 적을 수도 많을수도
* 향후 추가될지도 모르는 외부 시스템을 고려
* 누락되거나 유실된 데이터는 재처리 하지 않고 API 호출하여 동기화

등등 요구사항을 정리하고.......

기존에 내부적으로 로그 수집을 위하여 사용하고 있던 ES시스템[^1] 을 이용하면 서비스에 수정을 하지 않아도 능동적 처리가 가능할거 같아 이를 이용하기로 함

* Scenario
자사 서비스를 이용하고 있던 A회원이 ㅇ사 ㅁ서비스를 가입하고 이 시점부터 데이터의 동기화가 이뤄저야 한다. 첫 가입시점에서 A회원의 자사 데이터를 ㅇ사로 넘겨주
고
이후 자사 서비스를 이용하여 발생된 사용 이력에 대하여 ㅇ사로 동기화 한다(ㅇ사에서 발생된 사용 이력 제외)
상호 시스템간 통신오류및 장애로 데이터의 싱크가 틀어진 경우 A회원의 데이터 전체를 동기화

![구성도](http://i.imgur.com/U8KZlRN.png)

서비스단에서 발생되는 json로그를 logstash로 수집하여 tcp로 output 하고
tcp서버에서 해당 데이터를 받아 동기화 대상인 데이터인지 분석 후 다시 외부 서버로 전송
전송할때도 마찬가지로 단일 API(json데이터) 규격으로 전달

**Logstash 5.2 : 데이터 수집**
**Netty 4.1.5.Final (tcp서버) : 데이터 처리 및 송/수신**

매우 단순한 구성으로 시스템을 구축하였지만 위 구성에서의 핵심은 로그를 수집하는 logstash가 아닐까 싶다
이전에는 내부적으로 서비스 로그를 남기지도 않았을 뿐더러 서비스에서 발생하는 데이터를 보기위해서는 무조건 DB를 조회하는 방법밖엔 없었는데 이미 정의된 scheme>의 데이터가 아니면 볼 방법조차 없었고 추가 하고자 하면 서비스에서부터 DB까지 모두 손봐야 하던 구조였지만, 로그 표준화 시스템을 적용하면서 이를 기반으로 다양>한 데이터의 사용이 가능해졌다.
이번 경우에는 ES를 직접적으로 사용하지는 않지만 logstash로 로그를 가져오고 사용한다는 점에서 내부적으로는 첫 ES 연계개발 사례라 볼 수 있겠다.

[^1]: 자체 개발한 표준화된 JSON로그 출력 라이브러리로 로그를 남기고 Logstash를 이용하여 Elasticsearch에 저장