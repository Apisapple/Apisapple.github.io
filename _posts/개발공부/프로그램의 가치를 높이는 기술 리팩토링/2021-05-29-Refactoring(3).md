---
published: true
comments: true
date: 2021-06-28 21:00 GMT+0900
title: 리팩토링(3)_method 정리
categories:
 - Refactoring
tags:
 - develop
 - Refactoring
---

# Refactoring (3)_method 정리 - 1st

### 1. 매서드 추출 (Extract Method)

* **어떤 코드를 그룹으로 묶어도 괜찮다고 판단이 된다면, 그 <u>코드를 빼내어 목적을 잘 나타내는 직관적인 이름의 method로 만들자.</u>**

#### 방법

* 목적에 부합하는 이름의 새로운 method를 생성하자. 이 때 이름은 원리가 아닌 기능을 나타내는 이름으로 한다.
  * method로 빼낼 코드가 한 줄의 명령이나 함수 호출 같이 아주 간단한 것이라면 새 method명을 통해 그 코드의 기능(목적)을 더 잘 드러낼 수 있을 때만 추출을 실시하자. **더 이해하기 쉬운 이름으로 추출하지 못할 경우 코드를 추출하지 말아야 한다.**
* 기존 method에서 빼낸 코드를 새로 생성한 method로 복사하자
* 빼낸 코드에서 기존 method의 모든 지역변수 참조를 찾자. 
  그것들을 새로 생성한 method의 지역변수나 매개변수로 사용할 것이다.
* 빼낸 코드 안에서만 사용되는 임시변수가 있는지 파악해서 있다면 그것들을 새로 생성한 method안에 임시변수로 선언
* 추출 코드에 의해 변경되는 지역변수가 있는지 파악하자. 
  만약 하나의 지역변수만 변경된다면 추출 코드를 method 호출처럼 취급할 수 있는지 알아내고, 그 결과를 변수에 대입할 수 있는지 알아내자. 이렇게 하기 까다롭거나 둘 이상의 지역변수가 변경될 때는 method를 추출하기 위해 먼저 임시변수 분리 등의 기법을 적용해야 할 수 있다. 임시변수를 제거하려면 <u>임시변수를 method 호출로 전환 기법</u>을 적용하면 된다.
* 빼낸 코드에서 읽어들인 지역변수를 대상 method에 매개변수로 전달한다.
* 모든 지역변수 처리를 완료했으면 컴파일을 실시하자
* 원본 method안에 있는 빼낸 코드 부분을 새로 생성한 method 호출로 수정하자
* 컴파일 및 테스트를 진행한다.



### 2. 매서드 내용 직접 삽입 (Inline Method)

* **<u>method 기능이 너무 단순해서 매서드명만 봐도 너무 뻔한 내용</u>일 때,** 
  **그 method의 기능을 호출하는 method에 내용을 넣고 그 메서드는 삭제하자**



#### 방법

* 메서드가 재정의되어 있지 않은지 확인
  * 그 메서드가 하위클래스에 재정의되어 있다면 없어진 매서드를 재정의하는 일이 생겨서는 안되기 때문에 메서드 내용 직접 삽입을 실시하지 않는 것이 좋다.
* 그 메서드를 호출하는 부분을 모두 확인
* 각 호출 부분을 메서드 내용으로 교체
* 테스트 진행
* 정의된 메서드를 삭제



### 3. 임시변수 내용 직접 삽입 (Inline Temp)

* **간단한 수식을 대입받는 임시변수로 인해 다른 리팩토링 기법 적용이 힘들 땐,**
  **그 임시변수를 참조하는 부분을 전부 수식으로 치환하자**

```java
// 간단한 예시

/* 리팩토링 이전 */
double totalPrice = Order.totalPrice();
return (10000 < totalPrice);

/* 리팩토링 이후 */
return (10000 < Order.totalPrice());
```



#### 방법

* 대입문의 우변에 문제가 없는지 확인한다
* 문제가 없다면 임시변수를 final로 선언하고 컴파일을 진행
  * final로 선언함으로서 임시변수에는 값을 한 번만 대입할 수 있다.
* 그 임시변수를 참조하는 모든 부분을 찾아서 대입문 우변의 수식으로 변환
* 하나씩 수정을 할 때마다 컴파일 및 테스트 진행
* 임시변수 선언과 대입문을 삭제
* 컴파일 및 테스트 진행



### 4. 임시변수를 매서드 호출로 전환 (Replace Temp with Query)

* **수식의 결과를 저장하는 임시변수가 있을 땐,**
  **그 수식을 빼내어 메서드로 만든 후, 임시변수 참조 부분을 전부 수식으로 교체하자.**
  **새로 만든 메서드는 다른 메서드에서도 호출이 가능하다.**



```Java
/* 리팩토링 이전 */
double basePrice = itemCnt * itemPrice;
if(basePrice > 10000)
    return basePrice * 0.95;
else
    return basePrice * 0.98;



/* 리팩토링 이후 */
double basePrice() {
	return itemCnt * itemPrice;
}
if(basePrice() > 10000)
    return basePrice() * 0.95;
else
    return basePrice() * 0.98;
```



#### 동기

* 임시변수는 일시적이고 적용이 국소적인 범위로 제한이 된다. 임시변수는 자신이 속한 메서드의 안에서만
  인식되므로, 그 임시변수에 접근하려면 코드는 길어지게 된다.
  이럴 때, 임시변수를 매서드 호출로 수정하면 클래스 안 모든 메서드가 그 정보에 접근이 쉬워진다.



#### 방법

* 값이 한 번만 대입되는 임시변수를 확인
  * 값이 여러 번 대입되는 임시변수가 있다면 임시변수 분리 기법 실시를 고려해야 한다.
* 그 임시변수를 final로 선언
  * 임시변수엔 값을 한 번만 대입할 수 있다.
* 컴파일을 실시
* 대입문 우변을 빼내어 메서드로 변환
  * 처음엔 메서드를 private로 선언. 그 후 더 여러 곳에서 사용하게 되면 접근 제한을 완화한다.
  * 추출 메서드에 문제는 없는지 확인을 해야한다. 
    만약 <u>**객체 변경 등의 문제가 있다면 상태 변경 메서드와 값 반환 메서드를 분리 기법을 실시.**</u>
* 컴파일 및 테스트 진행
* 임시변수를 대상으로 임시변수 내용을 직접 삽입 기법을 실시



------

> **참고자료**
>
> - 리팩토링_코드 품질을 개선하는 객체지향 사고법 _마틴파울러 저