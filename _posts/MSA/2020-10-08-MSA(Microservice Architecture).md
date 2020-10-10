---
published: true
comments: true
date: 2020-10-10 16:00 GMT+0900
title: MSA(Microservice Architecture)
categories:
 - Architecture
tags: 
 - Architecture
 - Micro Service Architecture
---



# MSA(Microservice Architecture)

> 하나의 큰 어플리케이션을 여러개의 작은 어플리케이션으로 쪼개어 변경과 조합이 가능하도록 만든 아키텍쳐

## Monolithic 과  Microservice

### [Monolithic Architecture](https://ahea.wordpress.com/2018/04/26/msa-1-monolithic-architecture-%EB%9E%80/)

- 장점
    1. 어떤 기능이든지 개발되어있는 환경이 같아 복잡하지 않다.
    2. 쉽게 높은 가용성을 가진 서버 환경을 만들 수 있다.
    3. [End-to-End](https://medium.com/hbsmith/e2e-test-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-3c524862469d) 테스트가 용이하다.

- 단점
    1. 한 프로젝트의 크기가 너무 커지게 될 경우, 구동시간이 늘어나고 
    빌드, 배포하는 시간이 길어진다.
    2. 작은 수정사항이 있어도 전체를 다시 빌드하고 배포해야한다.
    3. 코드의 양이 몰려있어 개발자가 모두 이해 할 수 없고 유지보수가 어렵다.
    4. 일부분의 오류가 전체 서비스에 영향을 미친다.
    5. 기능별로 기술, 언어, 프레임 워크를 선택하기 어렵다.

### [Microservice Architecture](https://ko.wikipedia.org/wiki/%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4)

- 장점
    1. 기능별로 서비스를 개발하고, 작업 할당을 서비스 단위로 하기 떄문에 개발자가 해당 부분을 온전히 이해할 수 있다.
    2. 기능을 추가하거나 수정사항이 있는 서비스만 빠르게 빌드,
    배포가 가능하다.
    3. 해당 기능에 맞는 기술, 언어 등을 선택하여 사용할 수 있다.
    4. 일부분의 오류가 있으면 해당 기능에만 오류가 발생하고
    그 부분만 수정하여 빠른 정정화가 가능하다.
- 단점
    1. 관리가 어렵다. 서비스가 분산되어 모니터링이 힘들다.
    2. 서로를 호출하여 전체 서비스를 구성하기 때문에 다른 서비스를 호출하는 코드가 추가된다. (개발이 까다롭다.)
    3. 통신과 관련된 오류가 잦을 수 있다.
    4. 테스트가 불편하다. 
    (테스트를 위해 여러 서비스를 동작시켜야 한다.)

전체 서비스를 작은 단위로 나누어 개발하고 합쳐서 동작시킨다는 것이 
학과 수업 중 **프로그래밍 구조와 설계**에서 들은 내용이 생각나는 아이디어였다.

## MSA의 종류

---

### 1. Inner Architecture

> 내부 서비스와 관련된 Architecture
내부의 서비스를 어떻게 잘 쪼개는지에 대한 설계

- Inner Architecture 설계 시 고려사항
    - 서비스를 어떻게 정의할 것인가?
    - DB Access 구조를 어떻게 설계할 것인가?
    - 서비스 내 API를 어떻게 설계할 것인가?
    - 논리적인 컴포넌트들의 layer를 어떠한 방식으로 설계할 것인가?

### 2. Outer Architecture

### Outer Architecture의 6가지 요소

- External Gateway
    - 전체 서비스 외부로부터 들어오는 접근을 내부 구조를 드러내지 않고 처리하기 위한 요소
    - 사용자 인증과 권한 정책관리를 수행하며, API Gateway가 여기서 가장 핵심적인 역할을 담당
- Service Mesh
    - 마이크로서비스 구성 요소간의 네트워크를 제어하는 역할
    - Service discovery, Service routing, 트래픽 관리 및 보안 등을 담당하는 요소

- Container Management
    - 컨테이너 기반 어플리케이션 운영은 유연성과 자율성을 가지며, 개발자가 접근 및 운영할 수 있는 인프라 관리 기술이기 때문에 MSA에 적합
    - Kubernetes가 많이 사용된다.
- Backing Service
    - 어플리케이션이 실행되는 가운데 네트워크를 통해서 사용할 수 있는 서비스
    - 어플리케이션과 통신하는 attached Resource들을 지칭하는 포괄적인 개념
- Telemetry
    - 서비스들을 모니터링하고, 서비스별로 발생하는 이슈들에 대응할 수 있도록 환경을 구성하는 역할을 합니다.
- CI / CD Automation
    - 개발 단계를 자동화하여, 어플리케이션을 보다 짧은 주기로 
    고객에게 제공하는 방법
    - 지속적인 배포를 자동화 하는 것은 MSA 시스템에 꼭 필요한
    요소 중 하나