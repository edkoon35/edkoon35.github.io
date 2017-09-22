---
title: Elasticsearch 형태소 분석기 은전한잎 플러그인 설치
date: 2017-08-03 11:53:53
tags:
- elasticsearch
- eunjeon
- 형태소분석기
---
# Elasticsearch에 한글 형태소 분석기 은전한잎 설치
[nacyot님의 블로그][blog] 글을 참고하였으며, 5.5.1 버전에 맞게끔 텍스트를 수정하였습니다.

``설치시 root로 진행``
# 은전한잎(mecab-ko) 설치 ([Docs][1])
```
# mecab-ko 다운로드
$ cd /opt
$ wget https://bitbucket.org/eunjeon/mecab-ko/downloads/mecab-0.996-ko-0.9.2.tar.gz
$ tar xvf mecab-0.996-ko-0.9.2.tar.gz

# 빌드 및 설치
$ cd /opt/mecab-0.996-ko-0.9.2
$ ./configure
$ make
$ make check
$ make install
$ ldconfig
```
설치시 저는 `yum install automake` 팩키지를 추가로 설치해 주었습니다.

# mecab-ko-dic 설치 ([Docs][2])
```
# mecab-ko-dic 다운로드
$ cd /opt
$ wget https://bitbucket.org/eunjeon/mecab-ko-dic/downloads/mecab-ko-dic-2.0.1-20150920.tar.gz
$ tar xvf mecab-ko-dic-2.0.1-20150920.tar.gz

# 빌드 및 설치
$ cd /opt/mecab-ko-dic-1.6.1-20140814
$ ./autogen.sh
$ ./configure
$ make
$ make install
```

# mecab-java 설치 ([Docs][3])
```
# 환경 변수 설정
$ export JAVA_TOOL_OPTIONS -Dfile.encoding=UTF8

# mecab-java 다운로드
$ cd /opt
$ wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=0B4y35FiV1wh7NHo1bEJxd0RnSzg' -O mecab-java-0.996.tar.gz
$ tar xvf mecab-java-0.996.tar.gz

# 빌드 및 설치
$ cd /opt/mecab-java-0.996
$ sed -i 's|/usr/lib/jvm/java-6-openjdk/include|/usr/lib/jvm/java-8-oracle/include|' Makefile
$ make

# 빌드된 파일 이동(elasticsearch 실행시 참조해주어야 함)
$ cp libMeCab.so /usr/local/lib
```

# 엘라스틱서치 mecab-ko 플러그인 설치 ([Docs][4])
```
# mecab-ko-lucene-analyzer 다운로드 (5.1.1.0 버전 이후부터는 버전에 맞게끔 자동으로 다운로드 하는 스크립트를 통해 다운로드함)

# download plugin
$ bash <(curl -s https://bitbucket.org/eunjeon/seunjeon/raw/master/elasticsearch/scripts/downloader.sh) -e <es-version> -p <plugin-version>

# install plugin
$ ./bin/elasticsearch-plugin install file://path/elasticsearch-analysis-seunjeon-<plugin-version>.zip
```
제 기준으로 `<es-version>`는 5.5.1 `<plugin-version>`는 5.1.1.0 으로 입력하였습니다.

# Elasticsearch 기본 한글 형태소 문장 검색
```
curl -XGET http://0.0.0.0:9200/_analyze?pretty=true -d '아버지가 방에 들어간다.'

# 결과  
{
  "tokens" : [
    {
      "token" : "아버지가",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "<HANGUL>",
      "position" : 0
    },
    {
      "token" : "방에",
      "start_offset" : 5,
      "end_offset" : 7,
      "type" : "<HANGUL>",
      "position" : 1
    },
    {
      "token" : "들어간다",
      "start_offset" : 8,
      "end_offset" : 12,
      "type" : "<HANGUL>",
      "position" : 2
    }
  ]
}
```

기본검색 기능을 이용하게 되면 위처럼 `아버지가` 통째로 분석되므로 `아버지`로는 검색을 할 수가 없다.
```
# 데이터 입력
$ curl -XPUT 'http://0.0.0.0:9200/default/text/1' -d '{"text": "아버지가 방에 들어간다"}'

# '아버지'로 검색
$ curl -XGET 'http://0.0.0.0:9200/default/_search?pretty=true' -d '{"query":{"term": {"text": "아버지"}}}}'

{
  "took" : 7,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 0,
    "max_score" : null,
    "hits" : [ ]
  }
}

# '아버지가'로 검색
$ curl -XGET 'http://0.0.0.0:9200/default/_search?pretty=true' -d '{"query":{"term": {"text": "아버지가"}}}}'

{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.25316024,
    "hits" : [
      {
        "_index" : "default",
        "_type" : "text",
        "_id" : "1",
        "_score" : 0.25316024,
        "_source" : {
          "text" : "아버지가 방에 들어간다"
        }
      }
    ]
  }
}
```
이번에는 korean이라는 이름으로 mecab_ko_standard_tokenizer가 적용된 인덱스를 생성한다.
```
$ curl -XPUT http://0.0.0.0:9200/korean/ -d '{
  "settings" : {
    "index":{
      "analysis":{
        "analyzer":{
          "korean":{
            "type":"custom",
            "tokenizer":"mecab_ko_standard_tokenizer"
          }
        }
      }
    }
  },
  "mappings": {
    "text" : {
      "properties" : {
        "text" : {
          "type" : "string",
          "analyzer": "korean"
        }
      }
    }
  }
}'
```
위 코드를 실행하면  
~~~
# 아래와 같이 해당 토크나이저를 찾을 수 없다고 에러가 똭! 발생하니 tokeninze를 추가해 줍니다..

{
   "error":{
      "root_cause":[
         {
            "type":"remote_transport_exception",
            "reason":"[node-es-1][master_ip:9300][indices:admin/create]"
         }
      ],
      "type":"illegal_argument_exception",
      "reason":"Custom Analyzer [korean] failed to find tokenizer under name [mecab_ko_standard_tokenizer]"
   },
   "status":400
}
~~~

```
$ curl -XPUT http://0.0.0.0:9200/korean/ -d '{
  "settings" : {
    "index":{
      "analysis":{
        "analyzer":{
          "korean":{
            "type":"custom",
            "tokenizer":"mecab_ko_standard_tokenizer"
          }
        },
          "tokenizer": {
            "mecab_ko_standard_tokenizer": {
              "index_eojeol": "true",
              "index_poses": [
                "UNK",
                "EP",
                "I",
                "J",
                "M",
                "N",
                "SL",
                "SH",
                "SN",
                "VCP",
                "XP",
                "XS",
                "XR"
              ],
              "decompound": "true",
              "type": "seunjeon_tokenizer"
            }
          }
      }
    }
  },
  "mappings": {
    "text" : {
      "properties" : {
        "text" : {
          "type" : "string",
          "analyzer": "korean"
        }
      }
    }
  }
}'
```


위에서 만든 인덱스를 통해서 한국어 문장을 다시 분석해 봅니다.
```
$ curl -XGET http://0.0.0.0:9200/korean/_analyze?analyzer=korean\&pretty=true -d '아버지가 방에 들어간다'

{
  "tokens" : [
    {
      "token" : "아버지/N",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "N",
      "position" : 0
    },
    {
      "token" : "아버지가/EOJ",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "EOJ",
      "position" : 0
    },
    {
      "token" : "방/N",
      "start_offset" : 5,
      "end_offset" : 6,
      "type" : "N",
      "position" : 1
    },
    {
      "token" : "방에/EOJ",
      "start_offset" : 5,
      "end_offset" : 7,
      "type" : "EOJ",
      "position" : 1
    },
    {
      "token" : "들어가/V",
      "start_offset" : 8,
      "end_offset" : 11,
      "type" : "V",
      "position" : 2
    },
    {
      "token" : "들어간다/EOJ",
      "start_offset" : 8,
      "end_offset" : 12,
      "type" : "EOJ",
      "position" : 2
    }
  ]
}
```
아버지 나 방 이 명사로 분석이 됨.
```
# 데이터 입력
$ curl -XPUT 'http://0.0.0.0:9200/korean/text/1' -d '{"text": "아버지가 방에 들어간다"}'  

# '아버지'로 검색
$ curl -XGET 'http://0.0.0.0:9200/korean/_search?pretty=true' -d '{"query":{"bool": {"must": [{"match": {"text":"아버지"}}]} }}'

{
  "took" : 17,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.33310556,
    "hits" : [
      {
        "_index" : "korean",
        "_type" : "text",
        "_id" : "1",
        "_score" : 0.33310556,
        "_source" : {
          "text" : "아버지가 방에 들어간다"
        }
      }
    ]
  }
}
```

[1]:https://bitbucket.org/eunjeon/mecab-ko
[2]:https://bitbucket.org/eunjeon/mecab-ko-dic
[3]:http://taku910.github.io/mecab/
[4]:http://eunjeon.blogspot.kr/2016/12/mecab-ko-lucene-analyzer-0210.html
[blog]:http://blog.nacyot.com/articles/2015-06-13-eunjeon-with-elasticsearch/
