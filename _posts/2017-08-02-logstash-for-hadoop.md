---
title: logstash webhdfs output issue.
date: 2017-08-02 11:48:13
tags:
---
Maybe you should increase retry_interval or reduce number of workers.
---

위 에러 메시지를 출력하며 logstash가 webhdfs로 파일을 동시에 파일에 쓸 수가 없어
데이터가 유실되는 현상이 발생하면 아래의 output 옵션 값들을 조절하여 해결할 수 있다.  
[출처][1]

```
input {
    kafka {
        bootstrap_servers => "host:9092"
        topics => ["test-topic"]
        codec => "json"
     }
}
filter {

}
output {
    webhdfs {
      workers => 1
      host => "host"
      port => 50070
      path => "/data/path"
      user => "hdfs"
      codec => "json"
      # flush_size => 500
      flush_size => 5000
      # idle_flush_time => 10
      idle_flush_time => 5
      # retry_interval => 3
      retry_interval => 3
    }
}
```
[1]: https://birdben.github.io/2017/02/07/Logstash/Logstash学习（七）Logstash的webhdfs插件/ "출처"
