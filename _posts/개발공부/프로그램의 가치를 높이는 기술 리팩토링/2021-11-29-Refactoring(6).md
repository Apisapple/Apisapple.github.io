---
published: false
comments: true
date: 2021-11-29 01:28 GMT+0900
title: 리팩토링(6)_객체 사이의 기능 이동
categories:
 - Refactoring
tags:
 - develop
 - Refactoring
---

## 클래스 내용 직접 삽입

클래스가 더이상 제 역할을 수행하지 못하여 존재할 이유가 없어졌을 때, 클래스에서 주어진 기능을 가장 많이 사용하는 다른 클래스에 클래스를 합친다.



### 방법

* 원본 클래스의  public 메서드를 합칠 클래스를 선언하고, 이 메서드를 전부 원본 클래스로 위임한다.
  * 원본 클래스의 메서드 대신 별도의 인터페이스가 좋겠다고 판단이 된다면, 클래스의 내용을 인터페이스 추출 기법을 사용한다.
* 원본 클래스의 모든 참조를 합칠 클래스 참조로 수정
  * 원본 클래스를 private로 선언하고 패키지 밖의 참조를 삭제한다. 
    그 후 컴파일러가 껍데기만 남은 원본 클래스를 참조를 찾아낼 수 있게 원본 클래스명을 변경한다.
* 컴파일 및 테스트 실시
* 원본 클래스 삭제



## 대리 객체 은폐



## 과잉 중계 매서드





> **참고자료**
>
> - 리팩토링_코드 품질을 개선하는 객체지향 사고법 _마틴파울러 저

