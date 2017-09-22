---
title: Elasticsearch 5.5.1 설치
date: 2017-08-02 18:20:57
tags:
- elasticsearch
- install
---
테스트 서버에 새롭게 설치해야하는 김에 Elasticsearch 설치 방법을 다시 정리해 본다.
---

Product: Elasticsearch
Version: 5.5.1 (GA RELEASE)
Release date: July 25, 2017

OS: CentOS release 6.8
MEM: 16G
CPU: 6core

jdk: 1.8

## 1. 설치파일 다운로드
```
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.5.1.tar.gz

$ tar zxf elasticsearch-5.5.1.tar.gz

$ ln -s elasticsearch-5.5.1 elasticsearch

$ cd elasticsearch
```
## 2. 설정파일 수정  
```
$ vi config/elasticsearch.yml
```

5.0 버전부터는 대부분의 설정을 클러스터 업데이트 API를 통해서 설정이 가능하며  
elasticsearch.yml 파일에는 node / cluster 설정을 하면 됩니다.  
여기에서는 간단한 설정만 바꿔주도록 하겠습니다.  
[중요한 설정정보 참고][1]
```
* cluster.name: es5-cluster
* path.data: /data/es5-cluster
* path.logs: /data/es5-logs
* node.name: ${HOSTNAME}
* bootstrap.memory_lock: true
* network.host: [ip1, _local_]
* discovery.zen.ping.unicast.hosts: ["ip1", "ip2", "ip3"]
* discovery.zen.minimum_master_nodes: 2
```

## 3. elasticsearch 데몬 실행
```
$ bin/elasticsearch -d
```
데몬 실행시 아래 오류가 발생하여 설정파일에 ``bootstrap.system_call_filter: false`` 를 추가함.[^link1]
```
ERROR: [1] bootstrap checks failed
[1]: system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk
```

## 4. 서비스 동작 확인
``$ curl localhost:9200``
```
{
  "name" : "test-server",
  "cluster_name" : "es5-cluster",
  "cluster_uuid" : "qOBzma4KQAqt82v9EmT0Aw",
  "version" : {
    "number" : "5.5.1",
    "build_hash" : "19c13d0",
    "build_date" : "2017-07-18T20:44:24.823Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.0"
  },
  "tagline" : "You Know, for Search"
}
```
[1]: https://www.elastic.co/guide/en/elasticsearch/reference/5.5/important-settings.html
[^link1]: https://www.elastic.co/guide/en/elasticsearch/reference/current/system-call-filter-check.html
