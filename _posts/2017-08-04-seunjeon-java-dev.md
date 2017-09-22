---
title: "seunjeon을 이용한 Java 형태소 분석기"
date: 2017-08-04 11:13:09
tags:
- seunjeon
- java
- 형태소분석기
---
# 목표
seunjeon을 이용하여 Java로 입맛에 맞는 형태소 분석기를 만들어 보겠습니다. ([seunjeon][1])  
`사전이 패키지 내에 포함되어 있기 때문에 별도로 mecab-ko-dic을 설치할 필요가 없습니다. 특징으로는 (시스템 사전에 등록되어 있는 단어에 한하여) 복합명사 분해와 활용어 원형 찾기가 가능합니다. (속도도 빨라염)` 라고 하니 안쓸 이유가 없습니다.!

# Gradle
```
compile group: 'org.bitbucket.eunjeon', name: 'seunjeon_2.11', version: '1.3.0'
```
# Java
```
import org.bitbucket.eunjeon.seunjeon.Analyzer;

class Smaple {
    public void main(String[] args) {
        // 형태소 분석
        for (LNode node : Analyzer.parseJava("아버지가방에들어가신다.")) {
            System.out.println(node);
        }

        // 어절 분석
        for (Eojeol eojeol: Analyzer.parseEojeolJava("아버지가방에들어가신다.")) {
            System.out.println(eojeol);
            for (LNode node: eojeol.nodesJava()) {
                System.out.println(node);
            }
        }

        /**
         * 사용자 사전 추가
         * surface,cost
         *   surface: 단어명. '+' 로 복합명사를 구성할 수 있다.
         *           '+'문자 자체를 사전에 등록하기 위해서는 '\+'로 입력. 예를 들어 'C\+\+'
         *   cost: 단어 출연 비용. 작을수록 출연할 확률이 높다.
         */
        Analyzer.setUserDict(Arrays.asList("덕후", "버카충,-100", "낄끼+빠빠,-100").iterator());
        for (LNode node : Analyzer.parseJava("덕후냄새가 난다.")) {
            System.out.println(node);
        }

        // 활용어 원형
        for (LNode node : Analyzer.parseJava("빨라짐")) {
            for (LNode node2: node.deInflectJava()) {
                System.out.println(node2);
            }
        }

        // 복합명사 분해
        for (LNode node : Analyzer.parseJava("낄끼빠빠")) {
            System.out.println(node);   // 낄끼빠빠
            for (LNode node2: node.deCompoundJava()) {
                System.out.println(node2);  // 낄끼+빠빠
            }
        }
    }
}
```

위의 기본 예제를 차후 기능을 추가하며 늘려가보도록 하겠습니다...

[1]:https://bitbucket.org/eunjeon/mecab-ko-dic
