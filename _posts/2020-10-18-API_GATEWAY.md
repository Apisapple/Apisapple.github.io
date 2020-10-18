---
published: true
comments: true
date: 2020-10-18 20:55 GMT+0900
title: API GATEWAY
categories:
 - Architecture
tags: 
 - Architecture
 - API Gateway
---

# API GATEWAY

> **MSA에서 언급되는 컴포넌트** 중 하나이며, 모든 **클라이언트 요청에 대한 End Point를 통합하는 서버**를 의미한다.

---

## API GATEWAY의 필요성

하나의 큰 서비스를 수십 혹은 수백개의 작은 서비스로 나누면서, 
이를 클라이언트에서 서비스를 직접 호출하는 형태라면 아래와 같은 문제가 생길 수 있다.

![assets/images/APIgateway/Base.png](/assets/images/APIgateway/Base.png)

- 각 서비스마다 인증 / 인가 등의 공통된 로직을 구현해야하는 번거로움이 발생한다.
- API호출을 기록하고 관리하기 어렵다.
- 클라이언트에서 여러 마이크로 서비스에 대한 번거로운 호출을 해야한다.1
- 내부 비즈니스 로직이 드러나게 되어 보안에 취약해진다.

따라서, 어느 정도 규모 이상의 MSA기반 어플리케이션에서는 모든 API서버들의 엔드포인트를 단일화 해주는 서버가 필요하게 되었고 다음과 같은 형태를 띄게 되었다.

![assets/images/APIgateway/API.png](/assets/images/APIgateway/API.png)

API Gateway는 API 서버 앞단에서 모든 API 서버들의 엔드포인트를 단일화 해주는 또다른 서버이다.
API에 대한 인증과 인가 기능을 가지고 있으며 메시지의 내용에 따라 라우팅하는 역할을 담당한다.

API GATEWAY는 ESB(Enterprise Service Bus)에서 시작하였다.
ESB는 SOAP/XML 웹서비스 기반의 구조, API GATEWAY는 JSON/REST 기반의 구조

---

## API GATEWAY의 주요 기능

### 인증 / 인가에 관련된 기능

> API GATEWAY의 API에 대한 인증과 인가와 관련된 기능이다. 
인증은, 
**API를 호출하는 클라이언트에 대한 identity(신분)를 확인해주는 기능**이고
인가는, 
**클라이언트가 API를 호출할 수 있는 권한이 있는지를 확인해주는 기능**이다.

- API 토큰 발급

인증과 인가를 거칠 때 마다 매번 사용자가 절차를 거치기엔 너무나도 불편하다. 
그래서 보통 토큰이라는 방식을 사용한다. 토큰방식은 사용자의 인가가 끝나면, 사용자가 API를 호출할 수 있는 토큰을 발급해준다. API서버는 이 토큰으로 사용자의 identity와 권한을 확인 후, API 호출을 허가해준다. 이 때, 클라이언트에 대한 인증은 API 직접적으로 API 게이트웨이가 하지 않고 뒷단에 있는 인증 서버가 이를 수행한다.

발급된 토큰은 API를 호출할 수 있는 권한 정보와 연관되어 있는데, 이 권한 정보를 토큰 자체에 저장하느냐 서버에 저장하느냐에 따라 2가지 종류로 나뉜다.

토큰 자체가 정보를 갖는 형태를 클레임 기반의 토큰(Claim based token)이라고 하며, JWT나 SAML 토큰 등이 이에 해당한다. 클레임 기반의 경우 토큰의 저장소가 필요없어 구현이 용이하지만, 토큰 자체에 클레임 정보가 들어있기 때문에 많은 양의 정보를 담기가 어렵다. 또한 한번 발급된 토큰은 변경이 어려워 토큰 통제가 어렵다는 단점이 있다. 따라서 클레임 기반의 토큰을 사용할 때는 토큰의 유효기간을 둬서  반드시 강제적으로 토큰을 주기적으로 재발급 받도록 해야한다.

클레임 기반의 토큰이 아닌 경우, 정보를 서버에 저장해놓게 되는데, 클라이언트로는 unique한 문자열만을 리턴해주는 경우이다. 서버 기반의 토큰이 일반적으로 많이 사용되는 형태인데, token에 연관되는 정보가 서버에 저장되기 때문에 안전하고, 많은 정보를 저장할 수 있으며 token에 대한 정보를 수정하기가 용이하다. 하지만 별도의 토큰 저장소를 유지해야 하기 때문에 구현 노력이 더 높게 든다. 토큰은 매 API 호출마다 정보를 가지고 와야하기 때문에 DBMS와 같은  FILE IO 기반의 저장소 보다는 redis, memcached와 같은 메모리 기반의 고속 스토리지를 사용하는 것이 좋다.

### API 라우팅 및 로드밸런싱

> 사용하는 서비스나 클라이언트에 따라서 다른 엔드포인트를 이용해서 
서비스를 제공하거나, 여러 개의 데이터 센터일 경우 데이터 센터간의 
라우팅 등을 지원하는 기능

- 백엔드 API 서버로의 로드 밸런싱

API gateway 뒷단에 다수의 API 서버가 있다고 할 때, 여러개의 API 서버로 부하를 분산하는 기능이 필요하다. 단순하게 Round Robin 방식으로 부하를 분산하는 기능뿐만 아니라, 각 서버 하드웨어에 따라 부하를 가중치를 줘서 분산하는 기능을 고려할 수 있다. 
또한 API 서버가 장애가 발생했을 때, 이를 감지해서 로드밸런싱 리스트에서 뺴고 복구 되었을 때, 다시 로드 밸런싱 기능에 넣는 기능들이 필요하다.
![assets/images/APIgateway/Routing.png](/assets/images/APIgateway/Routing.png)

   <출처 : [https://bcho.tistory.com/1005](https://bcho.tistory.com/1005)>

### 서비스 어그레게이션(Aggregation)

> 여러 개의 API를 묶어서 하나의 API로 만드는 작업

MSA에서 이러한 aggregation을 남발하는 경우 성능이 떨어져 버릴 수 있다. 이런 경우 무거운 aggreagtion 로직은 별도의 Mediator API 서버라는 계층을 만들어 API gateway 밖에서 따로 하는 방법을 사용할 수 있다.

### 로깅 및 미터링

API 호출 시 API 게이트웨이는 공통적으로 호출되는 부분인 만큼 모든 로그를 중간에서 수집하기 용이하다.

근래의 어플리케이션 아키텍쳐가 클라이언트와 서버간의 통신이 모두 API를 기반으로 하는 형태로 변경이 되어감에 따라 API 호출 패턴을 분석하면 사용자의 사용 패텅을 분석해낼 수 있다.

또한 API 호출 로그는 차후 문제가 발생하였을때, 문제를 추적하기 위한 중요한 자료로 사용된다.

## Reference

- [https://bcho.tistory.com/1005](https://bcho.tistory.com/1005)
- [https://meetup.toast.com/posts/201](https://meetup.toast.com/posts/201)
- [https://velog.io/@tedigom/MSA-제대로-이해하기-3API-Gateway-nvk2kf0zbj](https://velog.io/@tedigom/MSA-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-3API-Gateway-nvk2kf0zbj)