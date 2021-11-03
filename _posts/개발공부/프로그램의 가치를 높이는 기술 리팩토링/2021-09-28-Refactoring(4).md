---
published: true
comments: true
date: 2021-09-28 21:00 GMT+0900
title: 리팩토링(4)_method 정리_2
categories:
 - Refactoring
tags:
 - develop
 - Refactoring
---

# Refactoring (3)_method 정리 - 2nd

### 5. 직관적 임시변수 사용

* 사용된 수식이 복잡할 때, 수식의 결과나 수식의 일부부분을 용도에 맞는 직관적인 이름의 임시변수에 대입



#### 방법

1. 임시 변수를 final로 선언하고, 복잡한 수식에서 한 부분의 결과를 그 임시변수에 대입
2. 그 수식에서 한 부분의 결과를 그 임시변수의 값으로 교체
   * 수식의 결과 부분이 반복될 경우엔 한 번에 하나씩 교체
3. 컴파일 및 테스트 실시
4. 수식의 다른 부분을 대상으로 위의 과정을 반복 실시



### 6. 임시변수의 분리

* 루프 변수나 값 누적용 임시변수가 아닌 임시변수에 여러 번 값이 대입이 될 때는, 
  각 대입마다 다른 임시변수를 사용



#### 방법

1. 선언문과 첫 번째 대입문에 있는 임시변수 이름을 변경
2. 이름을 바꾼 새 임시변수를 final로 선언
3. 새로운 임시변수의 모든 참조 부분을 두 번째 대입문으로 수정
4. 두 번째 대입문에 있는 임시변수를 선언
5. 컴파일 및 테스트
6. 각 대입문마다 차례로 선언문에서 임시변수 이름을 변경하고, 그 다음 대입문까지 참조를 수정 
   위의 과정을 반복 수행



#### 예시

##### 리팩토링 전

```java
double getDistanceTrabelled (int time) {
	double result;
	double acc = primaryForce / mass;
	
	int primaryTime = Math.min(time, _dalay);
	result = 0.5 * acc * primaryTime * primaryTime;
	int secondaryTime = time - _delay;
	
	if(secondaryTime > 0) {
		double primaryVel = acc * _delay;
        // 변수 acc에 값이 2번 대입이 되는 상황 용도에 따라서 변경이 필요
		acc = (primaryForce + secondaryForce) / mass;
		result += primaryVel * secondaryTime + 
			0.5 * acc * secondaryTime * secondaryTime*
	}
	
	return result;
}
```



##### 리팩토링 후

```java
double getDistanceTrabelled (int time) {
	double result;
	final double primaryAcc = primaryForce / mass;
	
	int primaryTime = Math.min(time, _dalay);
	result = 0.5 * primaryAcc * primaryTime * primaryTime;
	int secondaryTime = time - _delay;
	
	if(secondaryTime > 0) {
		double primaryVel = primaryAcc * _delay;
        // 변수 acc에 2번 대입이 되는 것을 용도에 따라서 변경
		final double secondaryAcc = (primaryForce + secondaryForce) / mass;
		result += primaryVel * secondaryTime + 
			0.5 * secondaryAcc * secondaryTime * secondaryTime*
	}
	
	return result;
}
```



### 7. 매개변수로의 값 대입 제거

매개변수로 값을 대입하는 코드가 있을 때, 매개변수 대신 임시변수를 사용하게 수정



#### 방법

1. 매개변수 대신 사용할 임시변수를 선언
2. 매개변수로 값을 대입하는 코드 뒤에 나오는 매개변수 참조를 전부 임시변수로 수정
3. 매개변수로의 값 대입을 임시변수로의 값 대입으로 수정
4. 컴파일 및 테스트 실행



#### 예시

##### 리팩토링 전

```c++
 int discount(int inputVal, int quantity, int yearToDate) {
     if(inputVal > 50)
         inputVal -= 2;
     if(quantity > 100)
         inputVal -= 1;
     if(yearToDate > 10000)
         inputVal -= 4;
     
     return inputVal;
 }
```



##### 리팩토링 후

```c++
 int discount(int inputVal, int quantity, int yearToDate) {
     int result = inputVal;
     if(inputVal > 50)
         result -= 2;
     if(quantity > 100)
         result -= 1;
     if(yearToDate > 10000)
         result -= 4;
     
     return result;
 }
```



### 8. 메서드를 메서드 객체로 전환

지역변수 때문에 메서드 추출을 적용할 수 없는 긴 매서드가 있을 때,
그 메서드 자체를 객체로 전환해서 모든 지역변수를 객체의 필드로 전환. 
그 후,  그 메서드를 객체안의 여러 메서드로 쪼갠다.



#### 방법

1. 전환할 메서드의 이름과 같은 이름으로 새 클래스를 생성
2. 그 클래스에 원본 메서드가 들어 있던 객체를 나타내는  final 필드를 추가하고 
   원본 메서드 안의 각 임시변수와 매개변수에 해당하는 속성을 추가
3. 새 클래스에 원본 객체와 각 매개변수를 받는 생성자 메서드 작성
4. 새 클래스에 compute라는 이름의 메서드 작성
5. 원본 메서드 내용을 compute 메서드 안에 복사. 원본 객체에 있는 메서드를 호출할 때 원본 객체를 나타내는 필드를 사용
6. 컴파일 실행
7. 원본 메서드를 새 객체 생성과 compute 메서드 호출을 담당하는 메서드로 바꾼다.



### 9. 알고리즘 전환

알고리즘을 더 분명한 것으로 교체를 해야할 때, 해당 메서드의 내용을 새 알고리즘으로 변환



#### 방법

1. 교체할 간결한 알고리즘을 준비. 컴파일 실시
2. 새 알고리즘을 실행하면서 여러 번의 테스트 실시. 
   모든 테스트 결과가 같으면 알고리즘 전환이 성공한 것으로 판단.
3. 결과가 다르게 나온다면, 기존의 알고리즘으로 테스트 및 디버깅을 실시



------

> **참고자료**
>
> - 리팩토링_코드 품질을 개선하는 객체지향 사고법 _마틴파울러 저

