---
title: mecab 사용자 사전 추가하기
date: 2017-08-03 18:47:06
tags:
- elasticsearch
- mecab
- mecab-ko-dic
---
## Elasticsearch에서 사용할 mecab 사전을 추가해보자.
[내용출처][1]

기존에 한번도 사용되지 않았던 새로운 가수가 앨범을 발매하게 되면 기존 형태소 분석을 통해서는 분석이 안되는 경우가 발생합니다.
이럴때는 사용자 사전에 미등록된 가수명을 등록해주면 분석이 잘 될것 같으니 추가해 보도록 하겠습니다.

## 사용자 사전 위치
```
# 플러그인 개발을 위해 생성된 경로 아래에서 실행
$ cd mecab-ko-dic/user-dic
$ ll

total 16
-rw-r--r-- 1 mba mba   70 Mar  1  2015 nnp.csv
-rw-r--r-- 1 mba mba   40 Mar  1  2015 person.csv
-rw-r--r-- 1 mba mba  166 Mar  1  2015 place.csv
-rw-r--r-- 1 mba mba 1593 May 17  2015 README.md
```

## 사용자 사전 내용
```
$ cat person.csv

까비,,,,NNP,인명,F,까비,*,*,*,*,*
```

사전의 엔트리의 형식은 아래와 같다.
> 표층형/0/0/0/품사태그/의미분류/종성유무/읽기/타입/첫번째품사/마지막품사/표현

위 내용을 참조하여 가수이름 사용자 사전을 생성하였다.(singer.csv)
```
시아준수,,,,NNP,인명,F,시아준수,*,*,*,*,*
```
## 사용자 사전 빌드 ([Docs][2])
```
# 사용자 사전 추가
$ ./tools/add-userdic.sh

# 추가된 사전 확인(user- 가 추가됨)
$ cat user-singer.csv


시아준수,1788,3544,5482,NNP,인명,F,시아준수,*,*,*,*,*

# 추가된 파일을 사전으로 컴파일
$ make install
```

## mecab 로컬 테스트
```
$ mecab -d /usr/local/lib/mecab/dic/mecab-ko-dic

# 사전 추가전 조회 결과
시아준수
시	EP,*,F,시,*,*,*,*
아	EC,*,F,아,*,*,*,*
준	VX+ETM,*,T,준,Inflect,VX,ETM,주/VX/*+ᆫ/ETM/*
수	NNB,*,F,수,*,*,*,*

# 사전 추가후 조회 결과
시아준수
시아준수	NNP,인명,F,시아준수,*,*,*,*,*
```

사전이 추가되었으니 다시 elasticsearch 플러그인을 빌드하면 끝!!

### KIBANA test query

tokenizer옵션 [참고][3]

```
DELETE korean

PUT korean
{
  "settings" : {
    "index":{
      "analysis":{
        "analyzer":{
          "korean":{
            "type":"custom",
            "tokenizer":"seunjeon_default_tokenizer"
          }
        },
        "tokenizer": {
          "seunjeon_default_tokenizer": {
            "type": "seunjeon_tokenizer",
            "index_eojeol": true,
            "decompound": true,
            "index_poses": ["UNK", "EP", "E", "I", "J", "M", "N", "S", "SL", "SH", "SN", "V", "VCP", "XP", "XS", "XR"]
          }
        }
      }
    }
  },
  "mappings": {
    "text" : {
      "properties" : {
        "query" : {
          "type" : "text",
          "analyzer": "korean"
        }
      }
    }
  }
}

GET korean/_mapping

GET korean/_analyze
{
  "text": ["시아준수노래 틀어줘"],
  "analyzer": "korean"
}

PUT korean/text/1
{
  "query":"아이유노래 틀어줘"
}
PUT korean/text/2
{
  "query":"시아준수노래 도리안 그레이 틀어줘"
}
PUT korean/text/3
{
  "query":"노래 시아준수 도리안그레이틀어줘"
}

GET korean/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "query": "시아준수"
          }
        }
      ]
    }
  }
}
```


[1]:http://kugancity.tistory.com/entry/mecab에-사용자사전기분석-추가하기
[2]:https://bitbucket.org/eunjeon/mecab-ko-dic/src/ce04f82ab0083fb24e4e542e69d9e88a672c3325/final/user-dic/?at=master
[3]:https://bitbucket.org/eunjeon/seunjeon/raw/master/elasticsearch/
