---
title: elasticsearch-spark 추가 오류 발생시
date: 2017-08-07 15:53:43
tags:
- zeppelin
- hdp2.6
- es-hadoop
- elasticsearch
---
# ES-HADOOP On HDP Zeppelin
`HDP(2.6)` 위에 설치한 `Zeppelin(0.7.0.2.6.0.3-8)` 환경에서  
ES-HADOOP을 돌리고자 zeppelin spark2 dependency에 `org.elasticsearch:elasticsearch-spark-20_2.11:5.3.3`를 추가하였고  
실행을 하였더니 아래와 같은 오류가 발생함.
```
<console>:26: error: object elasticsearch is not a member of package org
       import org.elasticsearch.spark._
```

원인은 말 그대로 elasticsearch-spark jar를 찾을 수 없어서...  
zeppelin admin UI에서 추가한 dependency가 왜 안먹을까나;;; 이 이유가 중요한것 같은데 다른 방법으로 해결했다..

[es-hadoop jar 다운로드][1]

>zeppelin spark lib 경로에 직접 jar를 추가함  
>> $ pwd  
>> /usr/hdp/current/zeppelin-server/interpreter/spark/elasticsearch-spark-20_2.11-5.3.3.jar  


[1]:https://www.elastic.co/downloads/hadoop
