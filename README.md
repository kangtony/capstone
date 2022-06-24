# capstone
캡스톤 정리 - 4개조(뉴디지털코어, 연계, 마케팅, 구현)


### 단계별 수행목표

 # 계획 
 의 시스템 구현 모형은 아래와 같이 나타낼 수 있습니다.

Business Capability(중요도)에 따라 서비스를 분할하고, 각 서비스를 독립적으로 설계합니다. 임의의 서비스 추가나 삭제에도 운영 중인 기존 서비스가 받는 영향은 없으며, 장애 전파가 없는 격리된 구조로 구현됩니다.



API Gateway 는 유입되는 요청에 대해 인증 정보를 확인 후 각 마이크로서비스로 라우팅 합니다.

주문, 상품, 배송 서비스들은 중앙의 메시지 큐(Kafka)를 통해 이벤트 기반 Pub/Sub 방식으로 통신하되, 필요 시 REST 기반 직접 호출도 수행합니다.
이때, 여러 서비스의 복합 정보를 서비스 해야하는 '마이페이지’는, 여러 이벤트를 Subscribe 하여 데이터를 프로비저닝 합니다.
최종 시스템 구현을 위한 총 6 단계의 단계별 수행 목표는 다음과 같습니다.

분석단계
관심사 분리(Separation of Concern)에 따른 KPI 분석

12st 쇼핑몰의 성장에 따라, 조직에서 공유되던 KPI 가 분화되는 팀에 맞도록 분산 매핑되는 전이 과정 분석
코어(Core)와 서포팅(Supporting) 업무(조직)으로 나뉘어진 환경에서 운영상 발생할 Pain Point 분석
설계단계
MSAEasy 도구를 활용한 이벤트스토밍 기반 마이크로서비스 설계

책임의 분산, 디자인의 분산에 따른 팀별 서비스 설계
MSAEasy를 활용한 이벤트 트리븐 설계(이벤트스토밍) 및 서비스별 역학관계에 따른 컨텍스트 매핑
GCP(Google Cloud Platform) Outer Architecture 설계
구현단계
Spring-boot 기반 MSA Chassis 프레임워크를 활용한 서비스 구현

MSAEasy 도구로부터 MSA Template 코드 다운로드 및 서비스별 Business Logic 구현
API Gateway 및 토큰기반 인증체계 적용
이벤트 기반 Pub/Sub 채널을 통한 서비스간 커뮤니케이션
통합단계
EDA(Event Driven Architecture) 기반 분산된 마이크로서비스 통합

웹 컴포넌트를 활용한 MVVM 기반 Client-Side UI 통합
마이페이지 통합 View : CQRS(Command and Query Responsibility Segregation) 패턴을 적용한 데이터 프로젝션
Saga 패턴에 따른 Eventual Transaction & Rollback 구현
배포단계
CI/CD 파이프라인을 활용한 GCP 개발환경, 운영환경 자동 배포

소스 형상관리 서버와 GCP(Google Cloud Platform) Cloud Build 연동 및 파이프라인(Pipeline) 생성
클라우드 개발계, 운영계 자동 배포를 위한 배포 트리거 생성
특정 사용자(또는, 환경)를 대상으로 한 새로운 버전의 점진적 배포 전략(Canary deployment) 적용
운영단계
모니터링(Logging, Monitoring, Tracing)을 통한 운영

Google Stackdriver를 활용한 클라우드 모니터링 및 Alerting 환경설정
Metric 정보에 따른 모니터링 대쉬보드 커스터마이징
통합 로깅 및 분산 추적 (Distributed Tracing)

### 세분화 수준 
마이크로서비스 세분화 수준
MSA 서비스 구축 시, 범하기 쉬운 오류는 서비스를 왜 나누는지에 대한 기준과 어떻게 나눌 것인지에 대한 전략없이 나누는 개수에 목표를 두는 경우입니다.

나뉘어지는 서비스 개수를 KPI 실적으로 정하는 일은 더욱 있어서는 안됩니다.
나뉘어지는 서비스마다 조직과 매핑 되어 책임의 분산이 이루어져야 하고, Project 가 아닌 Product 관점에서 관리 되어야만 지속적인 DevOps 가 가능해 집니다.

중요한 것은, 서비스의 개수가 아니라 서비스의 굵기(세분성)로 이는, 초기 계획단계 시 고려되어야 합니다.

서비스 세분화 단위는 그 범주에 따라 다음 5 단계로 구분 됩니다.



Monolith
– 서비스를 나누지 않고 한 덩어리로 구현하는 경우
– 작고 빠르게 동작해야 하며, 변경이 자주 발생하지 않는 서비스는 모노리스 구조 적용
Mini Service
– 구현이 용이한 단위로, 점진적 MSA 전환이 쉬운 단위
Business capability
– 서비스의 중요도에 따른 분할로 팀 단위와 연결하기 좋은 경우
– 비즈니스적 성과 측정과 피버팅(Pivoting)이 용이한 단위
Bounded Context
– 비즈니스 도메인을 여러 서브 도메인으로 구분(Core, Supportive, General)하고 서브 도메인별 분할
– 커뮤니케이션이 원활한 단위 – 관심사 분리(Separation of Cencerns)가 용이한 경우
Aggregate
– 서비스 구분의 최소 단위
– ACID 가 준수되어야 하는 원자적 단위 – 서비스 유연성이 높은 경우 -
– * 참고로 12st 쇼핑몰은 분석에서부터 Business capability 단위로 서비스를 나누어 구현하고 있습니다.

### 구현 패턴 
구현 패턴
현존하는 다수의 MSA 기술 패턴들 중, 12st 쇼핑몰 구현 시 적용할 패턴은 다음과 같습니다.

REST 기반 서비스 호출 시, 장애 전파를 미연에 방지해 주는 서킷 브레이커(Circuit Breaker), 폴리글랏 퍼시스턴스, CQRS, MVVM 기반 UI 렌더링 등 패턴별 간단한 소개와 12st 쇼핑몰 구현 시, 적용 여부를 수립합니다.

구현 패턴	적용 시점 및 대상	12st 적용여부
Circuit Breaker	
성능저하를 막아주나, Fail-fast 전략으로 사용자 경험이 나빠질 수 있음

적용대상이 비동기, 이벤트 기반으로 처리가능하다면 그 기반으로 전환

적용
Database per service	
ACID 트랜잭션 비용을 포기

ACID 트랜잭션 비용 포기가 불가하다면, Shared Database 로 처리해야 하거나 Semantic Locking 통한 Eventual Transaction 상의 Lock 을 구현해야 함

적용
Service Registry	
서비스 레지스트리의 유형 선택: API 기반(Eureka), DNS 기반(Kube-dns)

DNS 기반 적용
Saga	
하나 이상의 마이크로서비스들에 걸친 트랜잭션이 필요한 경우, Database per service 패턴을 적용했을 때 유효함

마이크로서비스간 프로세스 실행시간이 상대적으로 길거나 예측하기 힘든 경우 (e.g. 결재), 비용이 높은 경우 (2PC 를 사용하기 힘든 상황)

적용
CQRS	
하나 이상의 마이크로서비스에서 추출한 데이터로 뷰를 구성해야 하는경우

잦고 빠른 마이크로서비스 내에서의 Read 가 발생하는 경우에 사용

적용
Event Sourcing	
이벤트 소싱은 비용이 높기 때문에 다음의 요구사항이 존재하는지 확인 필요: Undo 기능 등의 요구가 향후 생길 수 있는가?, 마이크로 서비스간의 폴리글랏 퍼시스턴스 요구?, 기능의 추가 잦음

이벤트 소싱에서의 이벤트는 Append only 이기 때문에 데이터의 Diff 의 정보를 충실히 포함해야 함

적용
Backends for frontends	
BFF는 매번 composite service 를 구현해야 하기 때문에 관련한 frontend 의 유형이 매우 다양한 경우는 가능한 API Gateway 의 기능을 사용하거나 RESTful, HATEOAS 를 사용 권고

미적용
API Gateway	
API Gateway의 유형이 다양하기 때문에 해당 기능과 역할에 따라, Service Mesh 혹은 기존 EAI (Camel library) 등에서 처리해야 하는 경우 발생할 수 있음.

적용
Client-side UI Composition	
Server-side Rendering 은 Microservice 의 장점을 희석하므로 가능한 MVVM 기반 Client-side Rendering 을 적용해야 함

적용




