---
title: (데이터베이스) 정규화(Normalization)와 역정규화(DeNormalization)
author: jun
date: 2021-08-30 16:47:00 +0900
categories: [Computer Science, Database]
tags: [cs, database]
comments: true
img_path: /assets/img/post/20230403/
---

## 정규화란?

관계형 데이터베이스(RDB)의 데이터 **중복을 최소화**하고 **이상현상을 방지**하기 위해 <u>데이터를 구조화하는 프로세스</u>를 정규화라고 합니다.

정규화란, 속성들 간의 **종속 관계**를 분석하여 **무결성을 유지**하면서 <u>다수의 릴레이션으로 분리하는 과정</u>이라고도 합니다.

> 이상현상이란?
>
> 불필요한 데이터 중복으로 인해 릴레이션에 대한 데이터 삽입, 수정, 삭제 연산을 수행할 때 발생할 수 있는 부작용을 의미합니다.
>
> 이상의 현상의 종류로는 세 가지가 있습니다.
>
> - 삽입 이상 : 새 데이터를 삽입하기 위해 불필요한 데이터도 함께 삽입해야 하는 문제
> - 갱신 이상 : 중복 튜플 중 일부만 변경하여 데이터가 불일치하게 되는 모순의 문제
> - 삭제 이상 : 튜플을 삭제하면 꼭 필요한 데이터까지 함께 삭제되는 데이터 손실의 문제

정규화를 통해 분해된 결과를 정규형이라 하며, 이 정규형의 종류는 <u>제1정규형, 제2정규형, 제3정규형, BCNF, 제4정규형, 제5정규형</u> 등이 있습니다.

## 함수 종속

**함수 종속**(FD, Functional Dependency)은 정규화 이론의 핵심입니다.

![image-20230403190054104](/image-20230403190054104.png)
_X가 Y를 함수적으로 결정한다._

함수 종속에 대한 예시를 살펴보겠습니다.

### 함수 종속 관계 예시 : 고객 릴레이션

| 고객아이디 | 고객이름 | 등급   |
| ---------- | -------- | ------ |
| apple      | 정소화   | gold   |
| banana     | 김선우   | vip    |
| carrot     | 신준석   | silver |

종속관계는 다음과 같습니다.

- 고객아이디 -> 고객이름
- 고객아이디 -> 등급
- 고객아이디 -> (고객이름, 등급)

함수 종속 관계 판단 시 유의 사항

- 속성 자체의 **특성과 의미를 기반**으로 함수 종속성을 판단해야 합니다.
  - 일반적으로 <u>기본키와 후보키는 릴레이션의 다른 모든 속성들을 함수적으로 결정</u>합니다. (일반적으로 **결정자의 역할**을 가집니다)
  - 기본키나 후보키가 아니여도 <u>다른 속성 값을 유일하게 결정하는 속성</u>은 함수 종속 관계에서 **결정자**가 될 수 있습니다.

- 함수 종속은 제2정규형부터 BCNF까지 적용됩니다.

### 부분 함수 종속 예시 : 고객 릴레이션

| 고객아이디 | 이벤트번호 | 당첨여부 | 고객이름 |
| ---------- | ---------- | -------- | -------- |
| apple      | E001       | Y        | 정소화   |
| apple      | E004       | N        | 정소화   |
| banana     | E002       | N        | 김선우   |
| carrot     | E003       | N        | 신준석   |

종속관계는 다음과 같습니다.

- 고객아이디 -> 고객이름
- {고객아이디, 이벤트번호} -> 당첨여부
- {고객아이디, 이벤트번호} -> 고객이름

해당 표에서, 고객이름은 {고객아이디, 이벤트번호}에서의 일부분인 고객아이디에 종속되어 있습니다.

**즉, 고객이름은 {고객아이디, 이벤트번호}에 부분 함수 종속** 됩니다.

![image-20230403190943773](/image-20230403190943773.png)
_부분 함수 종속_

### 완전 함수 종속 (FFD, Full Functional Dependency)

완전 함수 종속이란,

- 릴레이션에서 속성 집합 Y가 속성 집합 X에 함수적으로 종속되어 있지만, 속성 집합 X의 전체가 아닌 일부분에는 종속되지 않음을 의미합니다.
- 일반적으로 함수 종속은 완전 함수 종속을 의미합니다.
- 위 예시에서, 당첨여부는 {고객아이디, 이벤트번호}에 완전 함수 종속됩니다.

### 고려할 필요가 없는 함수 종속 관계

결정자와 종속자가 같거나, 결정자가 종속자를 포함하는 것처럼 당연한 함수 종속 관계는 고려하지 않습니다.

- 고객아이디 -> 고객아이디
- {고객아이디, 이벤트번호} -> 이벤트번호

## 정규형의 종류

정규형의 종류를 알아보기 전에, 먼저 **정규형**(NF, Normal Form)이란, <u>릴레이션이 정규화된 정도</u>를 의미합니다.

각 정규형마다 제약조건이 존재하는데, 정규형의 차수가 높아질수록 요구되는 제약조건이 많아지고 엄격해집니다. 따라서, 정규화를 할 때는 릴레이션의 특성을 고려해서 적합한 정규형을 선택하도록 합니다.

![image-20230403191654856](/image-20230403191654856.png)
_정규형의 종류_

### 제1정규형 (1NF, First Normal Form)

반드시, 제1정규형을 만족해야 관계형 데이터베이스의 릴레이션이 될 자격이 있습니다.

릴레이션에 속한 모든 속성의 도메인이 **원자 값**(atomic value)**으로만 구성**되어 있으면 제1정규형에 속합니다.

![image-20230403191902284](/image-20230403191902284.png)
_제1정규형_

바로 위의 표는, 제1정규형을 만족하지만 이상 현상이 발생하는 릴레이션의 예시입니다.

속성들의 종속관계를 살펴보면 다음과 같습니다. (각 <u>속성들의 특성과 의미를 기반</u>으로 함수 종속성을 판단해 봅니다)

- 고객아이디 -> 등급
- 고객아이디 -> 할인율
- 등급 -> 할인율
- {고객아이디, 이벤트번호} -> 당첨여부

위의 관계를 그림으로 표현해 보면 다음과 같습니다.

![image-20230403192222458](/image-20230403192222458.png)
_이상현상 발생_

이상 현상이 발생한 이유가 무엇일까요?

기본키인 {고객아이디, 이벤트번호}에 완전 함수 종속되지 못하고 **일부분인 고객아이디에 종속되는 등급과 할인율 속성이 존재**하기 때문입니다. <br/>즉, 등급, 할인율은 {고객아이디, 이벤트번호} 집합에서 **고객아이디 속성에 부분 함수 종속**됩니다.

해당 문제를 해결하기 위해서는, **부분 함수 종속이 제거**되도록 이벤트 참여 릴레이션을 분해해야 합니다. (분해된 릴레이션은 제2정규형에 속하게 됩니다)

### 제2정규형 (2NF, Second Normal Form)

제1정규형에 속하는 릴레이션이 제2정규형을 만족하기 위해서는 **부분 함수 종속을 제거**하고 <u>모든 속성이 기본키에 완전 함수 종속되도록 분해</u>해야 합니다.

> 릴레이션이 제1정규형이 속하고, 기본키가 아닌 모든 속성이 기본키에 완전 함수 종속되면 제2정규형에 속합니다.

![image-20230403193035505](/image-20230403193035505.png)
_제2정규형_

고객아이디 속성에 부분 함수 종속되어 있는 등급과 할인률을 분해합니다.

![image-20230403193603946](/image-20230403193603946.png)
_부분 함수 종속 제거_

분해된 릴레이션은 제2정규형을 만족하지만, 이상 현상이 발생합니다.

![image-20230403193731920](/image-20230403193731920.png)
_이상 현상 발생_

이상 현상이 발생하는 이유는, **이행적 함수 종속이 존재**하기 때문입니다.<br/>종속관계를 살펴보면 다음과 같습니다.

- 고객아이디(X) -> 등급(Y)
- 등급(Y) -> 할인율(Z)
- 고객아이디(X) -> 할인율(Z)

즉, 다음과 같이 그림으로 표현할 수 있습니다.

![image-20230403193906986](/image-20230403193906986.png)
_이행적 함수 종속_

> 이행적 함수 종속이란?
>
> 릴레이션을 구성하는 3개의 속성 집합 X, Y, Z에 대해 함수 종속 관계 X->Y와 Y->Z가 존재하면 논리적으로 X->Z가 성립되는데, 이때 Z가 X에 이행적으로 함수 종속되었다고 합니다.

문제를 해결하기 위해서는, 이행적 함수 종속이 제거되도록 고객 릴레이션을 분해해야 합니다.<br/>분해된 릴레이션은 제3정규형에 속하게 됩니다.

### 제3정규형 (3NF, Third Normal Form)

제2정규형에 속하는 릴레이션이 제3정규형을 만족하게 하려면 모든 속성이 기본키에 **이행적 함수 종속이 되지 않도록 분해**해야 합니다.

> 릴레이션이 제2정규형에 속하고, 기본키가 아닌 모든 속성이 기본키에 이행적 함수 종속이 되지 않으면 제3정규형에 속합니다.

![image-20230403194323513](/image-20230403194323513.png)
_이행적 함수 종속 제거_

이행적 함수 종속을 제거하기 위해 릴레이션을 분해합니다. 그림으로 표현하면 다음과 같습니다.

![image-20230403194449736](/image-20230403194449736.png)
_이행적 함수 종속 제거_

### 보이스/코드 정규형(BCNF, Boyce/Codd Normal Form)

하나의 릴레이션에 여러 개의 후보키가 존재하는 경우, 제3정규형까지 모두 만족해도 이상 현상이 발생할 수 있습니다.

강한 제3정규형(strong 3NF)이라고도 하며, 후보키를 여러 개 가지고 있는 릴레이션에 발생할 수 있는 이상 현상을 해결하기 위해 제3정규형보다 좀 더 엄격한 제약조건을 제시합니다.

보이스/코드 정규형에 속하는 모든 릴레이션은 제3정규형에 속하지만, 제3정규형에 속하는 모든 릴레이션이 보이스/코드 정규형에 속하는 것은 아닙니다.

다음의 그림은 제3정규형은 만족하지만, 보이스/코드 정규형을 만족하지 않습니다.

![image-20230403194708938](/image-20230403194708938.png)
_이상 현상 발생_

이상 현상이 발생하는 이유는 "담당강사번호"가 후보키가 아님에도, "인터넷강좌"속성을 결정하기 때문입니다.

![image-20230403194821603](/image-20230403194821603.png)
_이상 현상 발생_

이를 해결하기 위해서, 후보키가 아닌 결정자(인터넷강좌 속성)를 제거하여 릴레이션을 분해해야 합니다.

![image-20230403194930311](/image-20230403194930311.png)
_릴레이션 분해_

그림으로 표현하면 다음과 같습니다.

![image-20230403195012487](/image-20230403195012487.png)
_후보키가 아닌 결정자를 제거_

### 4정규형과 제5정규형

- 제4정규형
  - 릴레이션이 보이스/코드 정규형을 만족하면서, 함수 종속이 아닌 다치 종속(MVC, Multi Valued Dependecy)을 제거하면 제4정규형에 속합니다.
- 제5정규형
  - 릴레이션이 제4정규형을 만족하면서, 후보키를 통하지 않는 조인 종속(JD, Join Dependency)을 제거하면 제5정규형에 속합니다.

## 정규화 시 주의사항

모든 릴레이션이 제5정규형에 속해야만 바람직한 것은 아닙니다. 릴레이션 분해로 인한 연산(JOIN)이 많아져 응답 시간이 느려질 수도 있습니다.

일반적으로 제3정규형이나, 보이스/코드 정규형에 속하도록 릴레이션을 분해하여 데이터 중복을 줄이고 이상현상을 해결하는 경우가 많습니다.

## 정규화 과정 정리

![image-20230403200038221](/image-20230403200038221.png)
_정규화 과정_

## 역정규화란?

**역정규화란 정규화를 통해 분리되었던 릴레이션에서 중복을 허용하고 다시 통합하거나 분할하여 구조를 재조정하는 과정입니다.**

정규화된 릴레이션은 하나의 릴레이션을 분해하기 때문에 원하는 자료가 하나의 릴레이션에 존재하지 않아 조인을 해서 참조해야하는 상황이 잦습니다.

이는 데이터베이스에 저장된 자료를 검색하는 시간을 증가시키며 성능을 저하시킬 수 있습니다.

따라서, 데이터베이스의 물리적 설계 과정에서 성능을 향상시키기위해 역정규화를 실행합니다.
역정규화의 종류로는 릴레이션 역정규화와 속성 역정규화가 있습니다.

- 릴레이션 역정규화

  - 릴레이션의 역정규화에는 릴레이션을 병합하는 방법과 분할하는 방법이 있습니다.

    - 릴레이션 병합 : 두 릴레이션 간의 잦은 참조로 성능이 저하될 경우 이 문제점을 해결하기 위해 병합합니다.

    - 릴레이션 분할 : 릴레이션의 데이터를 검색할때는 목록중의 데이터를 순차적으로 조사하여 원하는 자료를 찾습니다.

      때문에, 자주 <u>사용하지 않는 속성이나 튜플이 릴레이션에 있을 경우</u> 검색시 성능을 저하하게 만듭니다.

      이 경우에는 **자주 사용하는 속성이나 튜플을 분해**하여 성능을 향상 시킬 수 있습니다.

      이 분할에는 **수직 분할**(자주 사용하는 속성과 그렇지 않는 속성을 구분해서 분할)과 
      **수평 분할**(자주 사용하는 튜플과 그렇지 않는 튜플을 구분해서 분할)이 있습니다.

- 속성 역정규화

  - 릴레이션의 성능을 향상시키기 위해 속성 또는 파생속성을 추가합니다.

    > **※파생 속성(Delivered Attribute)이란?** 
    >
    > 현재 릴레이션에는 없는 속성이지만 작업의 효율을 위해 힌 속성으로부터의 계산이나 가공에 의해 파생되는 속성을 의미합니다.

## 마침

> 궁금하시거나, 부족한 내용이 있다면 코멘트 부탁드립니다.  🙇🏻‍♂️

## 레퍼런스

[데이터베이스 정규화 배경지식 WIKI](https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4_%EC%A0%95%EA%B7%9C%ED%99%94#%EB%B0%B0%EA%B2%BD_%EC%A7%80%EC%8B%9D_:_%EC%A0%95%EC%9D%98)

<span style="color: grey">학부생 시절 수업 자료👀</span>
