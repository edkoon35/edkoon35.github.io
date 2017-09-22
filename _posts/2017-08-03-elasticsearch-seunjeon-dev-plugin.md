---
title: Elasticsearch 은전한잎 플러그인 개발
date: 2017-08-03 16:14:15
tags:
- elasticsearch
- 은전한잎
---
# Elasticsearch 에서 사용할 은전한잎 형태소 분석기 플러그인을 빌드해 보자.  
ES-version: 5.5.1  
seonjeon-version: 5.1.1.1

## git clone ([Docs][1])
```
$ git clone https://bitbucket.org/eunjeon/seunjeon/raw/master/elasticsearch/
$ cd elasticsearch/
```

## 플러그인 개발
```
$ ./scripts/download-dict.sh mecab-ko-dic-2.0.1-20150920

--2017-08-03 16:23:37--  https://bitbucket.org/eunjeon/mecab-ko-dic/downloads/mecab-ko-dic-2.0.1-20150920.tar.gz
Resolving bitbucket.org... 104.192.143.3, 104.192.143.2, 104.192.143.1, ...
Connecting to bitbucket.org|104.192.143.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://bbuseruploads.s3.amazonaws.com/a4fcd83e-34f1-454e-a6ac-c242c7d434d3/downloads/7f7f2725-cf12-4c23-8934-bec686cb3f93/mecab-ko-dic-2.0.1-20150920.tar.gz?Signature=hZFJo7tLPI2Ck8IoBK2Va8Wo%2BwA%3D&Expires=1501746817&AWSAccessKeyId=AKIAIQWXW6WLXMB5QZAQ&versionId=null&response-content-disposition=attachment%3B%20filename%3D%22mecab-ko-dic-2.0.1-20150920.tar.gz%22 [following]
--2017-08-03 16:23:38--  https://bbuseruploads.s3.amazonaws.com/a4fcd83e-34f1-454e-a6ac-c242c7d434d3/downloads/7f7f2725-cf12-4c23-8934-bec686cb3f93/mecab-ko-dic-2.0.1-20150920.tar.gz?Signature=hZFJo7tLPI2Ck8IoBK2Va8Wo%2BwA%3D&Expires=1501746817&AWSAccessKeyId=AKIAIQWXW6WLXMB5QZAQ&versionId=null&response-content-disposition=attachment%3B%20filename%3D%22mecab-ko-dic-2.0.1-20150920.tar.gz%22
Resolving bbuseruploads.s3.amazonaws.com... 52.216.18.248
Connecting to bbuseruploads.s3.amazonaws.com|52.216.18.248|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 48624930 (46M) [application/x-tar]
Saving to: `mecab-ko-dic-2.0.1-20150920.tar.gz'

100%[===============================================>] 48,624,930  5.34M/s   in 14s

2017-08-03 16:23:53 (3.22 MB/s) - `mecab-ko-dic-2.0.1-20150920.tar.gz' saved [48624930/48624930]

mecab-ko-dic-2.0.1-20150920/
mecab-ko-dic-2.0.1-20150920/configure.ac
mecab-ko-dic-2.0.1-20150920/Person-actor.csv
mecab-ko-dic-2.0.1-20150920/IC.csv
mecab-ko-dic-2.0.1-20150920/model.def
mecab-ko-dic-2.0.1-20150920/user-dic/
mecab-ko-dic-2.0.1-20150920/user-dic/place.csv
mecab-ko-dic-2.0.1-20150920/user-dic/README.md
mecab-ko-dic-2.0.1-20150920/user-dic/person.csv
mecab-ko-dic-2.0.1-20150920/user-dic/nnp.csv
mecab-ko-dic-2.0.1-20150920/Place.csv
mecab-ko-dic-2.0.1-20150920/rewrite.def
mecab-ko-dic-2.0.1-20150920/tools/
mecab-ko-dic-2.0.1-20150920/tools/add-userdic.sh
mecab-ko-dic-2.0.1-20150920/tools/mecab-bestn.sh
mecab-ko-dic-2.0.1-20150920/tools/convert_for_using_store.sh
mecab-ko-dic-2.0.1-20150920/right-id.def
mecab-ko-dic-2.0.1-20150920/VX.csv
mecab-ko-dic-2.0.1-20150920/XR.csv
mecab-ko-dic-2.0.1-20150920/NP.csv
mecab-ko-dic-2.0.1-20150920/XSN.csv
mecab-ko-dic-2.0.1-20150920/feature.def
mecab-ko-dic-2.0.1-20150920/README
mecab-ko-dic-2.0.1-20150920/pos-id.def
mecab-ko-dic-2.0.1-20150920/COPYING
mecab-ko-dic-2.0.1-20150920/ETM.csv
mecab-ko-dic-2.0.1-20150920/CoinedWord.csv
mecab-ko-dic-2.0.1-20150920/Symbol.csv
mecab-ko-dic-2.0.1-20150920/EC.csv
mecab-ko-dic-2.0.1-20150920/INSTALL
mecab-ko-dic-2.0.1-20150920/EF.csv
mecab-ko-dic-2.0.1-20150920/unk.def
mecab-ko-dic-2.0.1-20150920/J.csv
mecab-ko-dic-2.0.1-20150920/ETN.csv
mecab-ko-dic-2.0.1-20150920/NNBC.csv
mecab-ko-dic-2.0.1-20150920/XSV.csv
mecab-ko-dic-2.0.1-20150920/Group.csv
mecab-ko-dic-2.0.1-20150920/dicrc
mecab-ko-dic-2.0.1-20150920/VCP.csv
mecab-ko-dic-2.0.1-20150920/EP.csv
mecab-ko-dic-2.0.1-20150920/VV.csv
mecab-ko-dic-2.0.1-20150920/NNP.csv
mecab-ko-dic-2.0.1-20150920/NR.csv
mecab-ko-dic-2.0.1-20150920/Preanalysis.csv
mecab-ko-dic-2.0.1-20150920/Person.csv
mecab-ko-dic-2.0.1-20150920/left-id.def
mecab-ko-dic-2.0.1-20150920/Makefile.am
mecab-ko-dic-2.0.1-20150920/NorthKorea.csv
mecab-ko-dic-2.0.1-20150920/Inflect.csv
mecab-ko-dic-2.0.1-20150920/configure
mecab-ko-dic-2.0.1-20150920/VCN.csv
mecab-ko-dic-2.0.1-20150920/MAJ.csv
mecab-ko-dic-2.0.1-20150920/XSA.csv
mecab-ko-dic-2.0.1-20150920/Foreign.csv
mecab-ko-dic-2.0.1-20150920/NNB.csv
mecab-ko-dic-2.0.1-20150920/VA.csv
mecab-ko-dic-2.0.1-20150920/XPN.csv
mecab-ko-dic-2.0.1-20150920/clean
mecab-ko-dic-2.0.1-20150920/ChangeLog
mecab-ko-dic-2.0.1-20150920/install-sh
mecab-ko-dic-2.0.1-20150920/Wikipedia.csv
mecab-ko-dic-2.0.1-20150920/autogen.sh
mecab-ko-dic-2.0.1-20150920/Place-station.csv
mecab-ko-dic-2.0.1-20150920/Place-address.csv
mecab-ko-dic-2.0.1-20150920/AUTHORS
mecab-ko-dic-2.0.1-20150920/Hanja.csv
mecab-ko-dic-2.0.1-20150920/MM.csv
mecab-ko-dic-2.0.1-20150920/missing
mecab-ko-dic-2.0.1-20150920/.keep
mecab-ko-dic-2.0.1-20150920/char.def
mecab-ko-dic-2.0.1-20150920/matrix.def
mecab-ko-dic-2.0.1-20150920/NEWS
mecab-ko-dic-2.0.1-20150920/MAG.csv
mecab-ko-dic-2.0.1-20150920/NNG.csv
mecab-ko-dic-2.0.1-20150920/Makefile.in
```

## 사전 빌드
```
# 사전 빌드(mecab-ko-dic/* -> src/main/resources/*.dat)
# 사전빌드 명령어를 수행시키게 되면 5~10분정도 멈춰 있다가 다운로드가 되기 때문에 걱정하지 말고 기다리세요~

$ sbt -J-Xmx2G "run-main org.bitbucket.eunjeon.seunjeon.DictBuilder"

# sbt가 설치되어 있지 않다면 아래의 방법으로 설치 (centos)
$ su -
$ cd /tmp
$ wget http://dl.bintray.com/sbt/rpm/sbt-0.13.9.rpm
$ yum localinstll sbt-0.13.9.rpm
```

## esVerion 변경
```
# 빌드파일 수정

$ vi build.sbt

val esVersion = "5.1.1" --> 5.5.1
```

## zip 생성
```
$ sbt

[info] Loading project definition from /home/mba/apps/tmp/elasticsearch/project
[info] Set current project to seunjeon (in build file:/home/mba/apps/tmp/elasticsearch/)
> project elasticsearch
[info] Set current project to elasticsearch-analysis-seunjeon (in build file:/home/mba/apps/tmp/elasticsearch/)
> esZip
```

이후 elasticsearch/target 하위에 생성된 플러그인 파일 ``elasticsearch-analysis-seunjeon-assembly-5.5.1.1.zip`` 을 elasticsearch 에 설치하면 된다.


[1]:https://bitbucket.org/eunjeon/seunjeon/raw/master/elasticsearch/
