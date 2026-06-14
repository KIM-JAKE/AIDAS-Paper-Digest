# 기업 데이터에서 왜 온톨로지가 필요한가

## 0. 한 줄 결론

문서 chunk RAG는 "관련 문장을 찾는 시스템"에는 충분하지만, 기업 AI가 **객체, 상태, 관계, 제약, 이벤트, 비용, 리스크를 이해하고 다음 행동을 판단하는 월드모델**이 되려면 온톨로지가 필요하다.  

더 정확히 말하면 온톨로지는 LLM을 대체하는 것이 아니라, LLM이 기업 데이터를 읽고 행동할 때 참조하는 **명시적 의미 계층(semantic world model)** 이다.

---

## 1. 왜 "문서 다 때려넣기"만으로는 부족한가

### 1.1 Chunk RAG가 잘하는 것

문서 chunk RAG는 다음 문제에 강하다.

| 유형 | 예시 | chunk RAG 적합도 |
|---|---|---|
| 단순 질의응답 | "환불 규정 알려줘" | 높음 |
| 문서 요약 | "회의록 결정사항 뽑아줘" | 높음 |
| 근거 문장 검색 | "계약서에서 위약금 조항 찾아줘" | 높음 |
| FAQ/매뉴얼 검색 | "이 기능 설정 방법 알려줘" | 높음 |

RAG의 원래 강점도 여기에 있다. Lewis et al.의 RAG 논문은 외부 지식을 검색해 knowledge-intensive NLP 태스크의 factuality를 보강하는 접근을 제안했다. 즉 RAG는 기본적으로 **외부 지식 검색 + 생성** 문제에 강한 구조다.  
출처: [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks, NeurIPS 2020](https://papers.nips.cc/paper/2020/hash/6b493230205f780e1bc26945df7481e5-Abstract.html)

### 1.2 Chunk RAG가 약해지는 지점

기업 데이터에서는 질문이 단순히 "어느 문서에 뭐라고 쓰여 있나?"가 아니라 다음처럼 바뀐다.

- 이 고객이 어떤 상품군/세그먼트/계약 상태에 속하는가?
- 이 비용 항목은 어떤 프로젝트, 부서, 공급업체, 원가 드라이버에 귀속되는가?
- 이 장애 이벤트가 어떤 서비스, SLA, 고객군, 매출 리스크에 영향을 주는가?
- 이 업무 요청은 현재 정책과 권한 체계상 가능한가?
- 이 공정/업무/승인 흐름이 표준 절차를 위반했는가?
- 이 이벤트가 다음 달 비용, 재고, 납기, 인력 계획에 어떤 영향을 주는가?

이런 질문은 "문장 검색"이 아니라 **기업 세계의 상태와 관계를 해석하는 문제**다. 이때 chunk RAG만 쓰면 다음 문제가 생긴다.

| 한계 | 설명 |
|---|---|
| 개념 정규화 실패 | 같은 개념이 CRM, ERP, BI, 문서에서 서로 다른 이름으로 등장한다. |
| 관계 단절 | chunk 단위 검색은 고객-계약-제품-부서-비용-이벤트 관계를 유지하지 못한다. |
| multi-hop 추론 약함 | A가 B에 속하고 B가 C 정책을 따른다는 식의 간접 판단이 불안정하다. |
| 제약 검증 불가 | "가능/불가능", "위반/정상", "다음 단계 진행 가능"을 구조적으로 검증하기 어렵다. |
| 상태 변화 추적 약함 | 이벤트가 발생했을 때 기업 상태가 어떻게 바뀌는지 모델링하지 못한다. |
| 감사 가능성 부족 | 왜 그런 판단을 했는지 규칙/관계/근거 경로로 남기기 어렵다. |

---

## 2. 논문 근거: KG/온톨로지가 왜 필요한가

### 2.1 LLM과 KG는 상보적이다

Pan et al.의 로드맵 논문은 LLM과 Knowledge Graph를 결합하는 연구를 크게 세 갈래로 정리한다.

- KG-enhanced LLMs
- LLM-augmented KGs
- Synergized LLMs + KGs

핵심은 LLM은 자연어 처리와 생성에 강하고, KG/온톨로지는 **명시적 사실, 관계, 해석 가능성, 구조적 추론**에 강하다는 점이다.  
출처: [Unifying Large Language Models and Knowledge Graphs: A Roadmap](https://arxiv.org/abs/2306.08302)

기업용 표현으로 바꾸면:

> LLM은 비정형 입력 해석기이고, 온톨로지는 기업 세계의 구조이며, KG는 그 구조 위에 올라간 현재 상태/사실 그래프다.

### 2.2 기업 SQL QA에서 KG가 정확도를 크게 올린 실험

Sequeda, Allemang, Jacob의 기업 SQL QA 벤치마크는 특히 중요하다. 보험 도메인의 enterprise SQL schema, enterprise question, ontology/mapping 기반 KG를 비교했고, GPT-4 zero-shot으로 SQL DB에 직접 질문했을 때 정확도는 16%, KG 표현을 사용했을 때는 54%로 증가했다고 보고했다.  
출처: [A Benchmark to Understand the Role of Knowledge Graphs on Large Language Model's Accuracy for Question Answering on Enterprise SQL Databases](https://arxiv.org/abs/2311.07509)

이 논문의 의미:

- 기업 데이터 문제는 단순히 "테이블을 보여주면 LLM이 SQL을 잘 짜는가"가 아니다.
- LLM이 business concept, metric, entity, relationship, schema mapping을 이해해야 한다.
- 온톨로지/KG는 기업 DB 위에 올라가는 **business semantic layer** 역할을 한다.

셀링 포인트:

> 기업의 데이터베이스와 문서는 그대로는 AI가 이해하는 세계가 아니다. 온톨로지는 "Revenue", "Active Customer", "Churn", "Eligible Claim", "Critical Incident" 같은 비즈니스 개념을 실제 테이블/문서/이벤트와 연결하는 AI용 의미 계층이다.

### 2.3 GraphRAG는 chunk RAG의 global sensemaking 한계를 보완한다

Microsoft의 GraphRAG 논문은 전통적 RAG가 전체 코퍼스에 대한 global question, 예를 들어 "이 데이터셋의 주요 테마는 무엇인가?" 같은 질문에 약하다고 설명한다. GraphRAG는 문서에서 entity knowledge graph를 만들고, 관련 entity community를 요약해 전체 코퍼스 단위의 질의응답 성능을 높인다.  
출처: [From Local to Global: A Graph RAG Approach to Query-Focused Summarization](https://arxiv.org/abs/2404.16130)

이 논문이 주는 메시지:

- chunk RAG는 "질문과 비슷한 chunk"를 찾는다.
- GraphRAG는 "질문과 관련된 entity/relationship/community"를 찾는다.
- 기업 전략, 리스크, 비용, 고객 이슈처럼 문서 전체에 흩어진 관계를 요약해야 하는 질문은 graph index가 더 적합하다.

### 2.4 GraphRAG 서베이: 관계 구조가 복잡한 데이터에서는 graph retrieval 필요

GraphRAG survey는 RAG가 hallucination, domain-specific knowledge, outdated knowledge 문제를 완화하지만, entity 간 관계 구조가 복잡한 데이터베이스에서는 한계가 있다고 정리한다. GraphRAG는 entity 간 structural information을 활용해 더 정확하고 context-aware한 응답을 목표로 한다.  
출처: [Graph Retrieval-Augmented Generation: A Survey](https://arxiv.org/abs/2408.08921)

실무 해석:

> IR에서 검색 대상이 "문장"이면 chunk RAG, 검색 대상이 "관계 경로"이면 ontology/graph RAG다.

### 2.5 Ontology-grounded RAG: 도메인 온톨로지로 retrieval context를 구성

OG-RAG는 domain-specific ontology에 retrieval을 anchor해서 LLM context를 구성하는 방법을 제안한다. 논문은 기존 RAG가 structured domain knowledge를 충분히 반영하지 못해 suboptimal context generation을 만든다고 보고, ontology-grounded hypergraph를 사용해 entity relationship을 보존하는 retrieval을 제안한다.  
출처: [OG-RAG: Ontology-Grounded Retrieval-Augmented Generation for Large Language Models](https://arxiv.org/abs/2412.15235)

이 논문의 문장으로 방어 가능한 주장:

- 일반 RAG는 specialized knowledge에 적응하는 데 한계가 있다.
- 온톨로지는 entity와 interrelationship을 정의해 domain knowledge를 구조화한다.
- workflow나 predefined rules/procedures를 따라야 하는 decision-making task에서 특히 중요하다.

### 2.6 KG-based RAG: 필요한 subgraph를 가져와 reasoning

SubgraphRAG는 KG 기반 RAG에서 전체 문서나 전체 그래프가 아니라 질문에 필요한 subgraph를 retrieve하고, LLM이 그 위에서 reasoning하도록 한다. WebQSP, CWQ 등 KG QA 벤치마크에서 효율성, 정확도, grounding 개선을 보고한다.  
출처: [Simple Is Effective: The Roles of Graphs and Large Language Models in Knowledge-Graph-Based Retrieval-Augmented Generation](https://arxiv.org/abs/2410.20724)

실무 해석:

> 기업 AI가 필요한 것은 "더 긴 context"가 아니라 "질문에 필요한 관계 경로만 압축해 넣는 semantic context"다.

---

## 3. "온톨로지가 있어야만 팔 수 있는" IR 유즈케이스

아래는 온톨로지를 쓰지 않으면 불가능하다는 뜻이 아니라, **chunk RAG만으로는 안정적/감사 가능/반복 가능하게 구현하기 어렵고, 온톨로지가 제품 차별점이 되는 영역**이다.

### 3.1 Business metric QA

질문:

> "지난 분기 enterprise churn이 왜 올랐고, 어떤 고객군/제품/계약 조건이 영향을 줬나?"

chunk RAG는 관련 리포트와 회의록을 찾을 수 있다.  
온톨로지 기반 IR은 다음 관계를 따라간다.

```text
Customer -> Account -> Contract -> Product -> Plan
Contract -> RenewalEvent -> ChurnRisk
Product -> SupportTicket -> Incident
Incident -> SLAImpact -> RevenueRisk
```

가능해지는 기능:

- metric 정의 정규화
- 같은 고객/계약/제품의 cross-system 연결
- 영향 경로 탐색
- 원인 후보 ranking
- BI/문서/이벤트 로그 통합 답변

### 3.2 Cost prediction and simulation

질문:

> "A 공급업체 단가가 8% 오르면 Q3 gross margin과 프로젝트별 비용은 어떻게 변하나?"

chunk RAG는 관련 계약서/구매 리포트를 찾을 수 있다.  
온톨로지 기반 world model은 다음 구조를 가진다.

```text
Supplier -> Part/Service -> BOM/ServiceBundle -> Product/Project
Product -> CustomerSegment -> RevenueForecast
CostDriver -> CostModel -> MarginSimulation
```

가능해지는 기능:

- 비용 항목을 원가 드라이버에 매핑
- 공급업체/부품/프로젝트/매출 영향 경로 계산
- "what-if" 시뮬레이션
- 비용 변경 이벤트의 downstream impact 분석

### 3.3 Compliance and policy validation

질문:

> "이 데이터를 이 목적과 권한으로 해외 리전에 저장해도 되는가?"

chunk RAG는 개인정보 처리방침, 보안 정책, 법무 문서를 찾는다.  
온톨로지는 다음을 검증한다.

```text
EmailAddress is PersonalData
HealthRecord is SensitiveData
CrossBorderTransfer requires ExplicitConsent
MarketingPurpose requires OptIn
Intern lacks PayrollDataAccess
```

가능해지는 기능:

- 권한/목적/데이터 유형 기반 정책 판단
- 상위 개념을 통한 간접 위반 검출
- 규정 충돌 시 override 관계 적용
- decision trace 생성

### 3.4 Operational process conformance

질문:

> "이 업무 요청은 표준 승인 절차를 제대로 탔나?"

chunk RAG는 SOP/정책 문서를 찾는다.  
온톨로지 기반 시스템은 expected process와 observed event sequence를 비교한다.

```text
Expected: Request -> ManagerApproval -> SecurityReview -> Provisioning -> AuditLog
Observed: Request -> Provisioning
Missing: ManagerApproval, SecurityReview
Violation: UnauthorizedProvisioning
```

가능해지는 기능:

- 누락 단계 검출
- 비정상 transition 감지
- 다음 단계 진행 가능 여부 판단
- 감사 로그 자동 생성

### 3.5 Enterprise incident and root cause analysis

질문:

> "어제 결제 실패가 어떤 서비스, 고객, 매출, SLA에 영향을 줬나?"

chunk RAG는 장애 보고서와 Slack/티켓을 찾는다.  
온톨로지 기반 event graph는 다음을 연결한다.

```text
PaymentFailureEvent
  affects PaymentService
  causedBy DatabaseLatencySpike
  impacts EnterpriseCustomerGroup
  violates SLA-Checkout-99.9
  creates RevenueAtRisk
```

가능해지는 기능:

- 이벤트-서비스-고객-매출 영향 경로 추적
- 재발 이벤트 clustering
- root cause 후보 ranking
- 비용/보상/리스크 추정

### 3.6 Agent action validation

질문:

> "AI agent가 이 고객 계정에 환불/권한 변경/계약 수정 action을 해도 되는가?"

chunk RAG는 정책 문서를 찾을 수 있지만, action 실행 전 검증에는 약하다.  
온톨로지는 action schema와 constraint를 갖는다.

```text
RefundAction requires EligibleClaim
ContractModification requires LegalApproval
AdminAccessGrant requires SecurityApproval
HighValueCustomerEscalation requires AccountOwnerNotification
```

가능해지는 기능:

- tool/action 실행 전 permission check
- action precondition 검증
- 위험 action 차단
- audit trail 생성

---

## 4. 기업의 월드모델로서 온톨로지

### 4.1 월드모델의 정의

여기서 말하는 기업 월드모델은 단순 vision world model이 아니다. 기업 내 객체와 상태를 통합적으로 표현하는 모델이다.

```text
Objects: 고객, 계약, 제품, 자산, 장비, 서비스, 팀, 직원, 공급업체
States: 활성/비활성, 승인/미승인, 장애/정상, 위험/정상, 납기 지연, 비용 초과
Relations: owns, uses, dependsOn, reportsTo, supplies, governedBy, impacts
Events: 주문, 승인, 장애, 변경, 배포, 결제 실패, 비용 증가, 계약 갱신
Constraints: 권한, 정책, SLA, 예산, 규정, 선후행 조건
Models: 비용 모델, 수요 모델, 리스크 모델, 시뮬레이션 모델
```

### 4.2 왜 온톨로지가 월드모델의 뼈대인가

| 월드모델 기능 | 온톨로지가 주는 것 |
|---|---|
| 객체 식별 | 같은 고객/제품/계약을 시스템마다 다른 이름에서 canonical entity로 통합 |
| 상태 표현 | 현재 상태와 상태 전이를 명시적으로 표현 |
| 관계 추론 | 고객-서비스-장애-매출 같은 indirect impact 경로 탐색 |
| 제약 검증 | action precondition, policy, SLA, budget constraint 검증 |
| 시뮬레이션 | 이벤트가 비용/재고/매출/리스크 상태를 어떻게 바꾸는지 모델링 |
| 설명 가능성 | 판단 경로를 entity-relation-rule trace로 남김 |

Digital Twin 연구에서도 비슷한 문제가 나온다. Singh et al.은 DT가 real-world product/process/system의 imitation이며, 데이터 variety, dynamics, lifecycle semantics를 다루기 위해 ontology model을 제안한다. 해당 논문은 ontology가 DT domain knowledge를 capture하고, DB query/manage에 필요한 최소 데이터 구조와 semantics를 제공한다고 설명한다.  
출처: [Data management for developing digital twin ontology model](https://journals.sagepub.com/doi/10.1177/0954405420978117)

기업 AI에서 이를 일반화하면:

> Digital twin이 물리 시스템의 semantic model이라면, enterprise ontology는 기업 운영 시스템의 semantic twin이다.

### 4.3 월드모델 아키텍처

```text
비정형 데이터
  - 문서, 회의록, 티켓, 이메일, SOP, 계약서

정형 데이터
  - CRM, ERP, HRIS, billing, warehouse, BI tables

이벤트 데이터
  - 로그, audit trail, workflow events, incident events, sensor/vision events

        ↓ entity/relation/event extraction + mapping

Enterprise Ontology
  - class: Customer, Contract, Product, Service, Asset, Role, Policy, Event
  - relation: owns, dependsOn, governedBy, causedBy, affects, requires
  - constraint: permission, SLA, budget, policy, process precondition

        ↓ instance population

Enterprise Knowledge Graph / World State
  - canonical entities
  - current states
  - event histories
  - provenance

        ↓ retrieval/reasoning/simulation

AI Applications
  - ontology-grounded IR
  - impact analysis
  - cost simulation
  - process conformance
  - compliance validation
  - agent action validation
```

---

## 5. 이벤트 온톨로지: 왜 별도로 중요한가

### 5.1 이벤트 온톨로지란

이벤트 온톨로지는 기업에서 발생하는 "무슨 일이 일어났는가"를 구조화하는 서브 모델이다.

기본 구조:

```text
Event
  type: Incident / Approval / PaymentFailure / Deployment / PriceChange / Order / Shipment
  actor: who initiated or caused it
  object: what entity was affected
  time: when it happened
  place/system: where it happened
  precondition: what state existed before
  effect: what state changed after
  cause: what caused it
  consequence: what it affected
  evidence: source logs/docs/tickets
```

### 5.2 이벤트 중심 모델이 필요한 이유

일반 KG는 entity 중심이기 쉽다.

```text
Customer A
Contract B
Product C
Service D
```

하지만 기업 운영은 event 중심으로 움직인다.

```text
Customer A renewed Contract B
Service D failed at 10:03
Supplier X increased price by 8%
Manager Y approved request Z
Agent Q changed permission P
```

EventKG 논문은 기존 DBpedia/YAGO/Wikidata 같은 KG가 entity 중심이라 event와 temporal relation coverage가 부족하다고 지적하고, event-centric temporal KG가 필요하다고 설명한다. EventKG는 69만 개 이상의 events와 230만 개 이상의 temporal relations를 통합했다.  
출처: [EventKG: A Multilingual Event-Centric Temporal Knowledge Graph](https://arxiv.org/abs/1804.04526)

### 5.3 이벤트 온톨로지의 핵심 기능

| 기능 | 설명 | 기업 유즈케이스 |
|---|---|---|
| 상태 전이 | event가 entity state를 어떻게 바꾸는지 표현 | 주문 생성, 결제 실패, 계약 갱신, 권한 변경 |
| 시간 추론 | before/after/during/overlap 관계 표현 | 장애 타임라인, SLA 위반, 승인 지연 |
| 원인-결과 | causedBy, leadsTo, affects 표현 | root cause analysis, impact analysis |
| 역할 표현 | actor가 event에서 어떤 role을 했는지 표현 | 승인자, 요청자, 담당자, 자동화 agent |
| provenance | 어떤 데이터/로그/사람이 근거인지 표현 | 감사, 컴플라이언스, 품질 검증 |
| simulation input | event가 모델의 입력으로 쓰임 | 비용 예측, 수요 예측, 리스크 시뮬레이션 |

### 5.4 SEM, PROV-O, OWL-Time과 연결

Simple Event Model(SEM)은 event-centered modeling이 도메인의 dynamic aspect를 포착하고, people/place/action/object 간 복잡한 관계를 명시하는 자연스러운 방법이라고 설명한다. 특히 "who did what, when and where"뿐 아니라 actor role, role validity time, authority/source를 표현해야 한다고 정리한다.  
출처: [Design and use of the Simple Event Model](https://www.sciencedirect.com/science/article/pii/S1570826811000199)

PROV-O는 Entity, Activity, Agent와 used/generated/wasDerivedFrom/wasAssociatedWith 같은 관계로 provenance chain을 구성한다. 이는 기업 AI가 "어떤 action이 어떤 데이터를 사용했고, 누가 책임지고, 어떤 결과를 만들었는가"를 추적하는 데 중요하다.  
출처: [PROV-O: The PROV Ontology](https://www.w3.org/TR/2013/PR-prov-o-20130312/)

OWL-Time은 temporal entity와 interval relation을 표준화하며, PROV-O의 Activity가 시간 구간을 갖는다는 점에서 자연스럽게 align된다.  
출처: [Time Ontology in OWL](https://www.w3.org/TR/owl-time/)

### 5.5 이벤트 온톨로지 유즈케이스

#### A. 비용 예측

```text
SupplierPriceIncreaseEvent
  affects MaterialCost
  affects ProductMargin
  affects ProjectBudget
```

효과:

- 단가 변경 이벤트가 어떤 제품/프로젝트/고객 계약에 영향을 주는지 계산
- 비용 모델의 입력으로 이벤트를 사용
- "왜 비용 예측이 바뀌었는가"를 설명 가능

#### B. 장애/리스크 시뮬레이션

```text
ServiceOutageEvent
  affects CheckoutService
  affects EnterpriseCustomerGroup
  violates SLA
  creates RevenueAtRisk
```

효과:

- 서비스 장애 이벤트를 고객/매출/SLA 영향으로 연결
- 비슷한 이벤트 재발 시 예상 피해 시뮬레이션
- 대응 우선순위 자동화

#### C. 업무 프로세스 감사

```text
AccessGrantedEvent
  actor ITAdmin
  object PayrollSystem
  missing SecurityApprovalEvent
  violates AccessControlPolicy
```

효과:

- 승인 누락 탐지
- 권한 변경 audit 자동화
- agent action 검증

#### D. Vision + enterprise world model 결합

```text
ObservedEvent: WorkerEnteredRestrictedArea
  actor Worker-17
  location Zone-A
  time 10:03
  relatedPolicy RestrictedAreaPolicy
  requiredRole AuthorizedTechnician
  observedRole Operator
```

효과:

- vision event를 단순 detection이 아니라 policy violation으로 해석
- 영상 세계와 기업 권한/작업/안전 규정 세계를 연결
- "무엇을 봤는가"에서 "기업적으로 무슨 의미인가"로 전환

---

## 6. IR에서 온톨로지를 써야 하는 경우 vs 안 써도 되는 경우

### 6.1 온톨로지를 써야 하는 IR

| 조건 | 왜 ontology가 필요한가 | 예시 |
|---|---|---|
| 질문이 entity 관계를 따라간다 | chunk similarity만으로 관계 경로가 보존되지 않음 | 고객-계약-제품-장애-매출 영향 |
| 같은 용어가 시스템마다 다르다 | canonical concept/alias가 필요 | ARR, recurring revenue, subscription revenue |
| 상하위 개념 추론이 필요하다 | 하위 개념을 상위 정책에 연결해야 함 | contractor is non-employee, non-employee lacks access |
| 제약/정책 검증이 필요하다 | 가능/불가능 판단은 검색이 아니라 validation | data transfer allowed? refund allowed? |
| 이벤트와 상태 변화가 중요하다 | 현재 상태는 문서가 아니라 event history에서 계산됨 | 계약 갱신됨, 권한 변경됨, 장애 발생 |
| 시뮬레이션/예측이 필요하다 | 모델 입력과 영향 경로가 구조화돼야 함 | 비용 상승 impact, SLA risk forecast |
| 감사 가능성이 중요하다 | 판단 경로를 규칙/관계/근거로 남겨야 함 | compliance, finance, security, HR |

### 6.2 온톨로지를 안 써도 되는 IR

| 조건 | 추천 방식 |
|---|---|
| 단순 문서 질의응답 | chunk RAG |
| 회의록/문서 요약 | chunk RAG + extractor |
| 근거 문장 찾기 | vector/BM25 hybrid search |
| flat FAQ | RAG |
| 일회성 리서치 | RAG + reranker |
| 관계/상태/제약이 적음 | ontology는 오버엔지니어링 |

### 6.3 실무 판단 공식

```text
문장 검색이면 RAG
관계 검색이면 GraphRAG
정책/상태/행동 검증이면 Ontology + KG
예측/시뮬레이션이면 Ontology + Event Model + Domain Model
```

---

## 7. 제품 셀링 포인트

### 7.1 안 좋은 포지셔닝

> "우리는 문서를 더 잘 검색합니다."

이건 vector DB/RAG 업체와 차별화가 약하다.

### 7.2 좋은 포지셔닝

> "우리는 기업의 데이터를 AI가 행동 가능한 월드모델로 바꿉니다."

또는:

> "문서 RAG가 SOP/정책/리포트를 찾아준다면, ontology-grounded AI는 현재 상황이 어떤 상태이고, 어떤 규칙을 위반했고, 어떤 비용/리스크/액션으로 이어지는지 판단합니다."

### 7.3 핵심 문장

- 문서 RAG는 evidence retrieval이고, 온톨로지는 enterprise state reasoning이다.
- 기업 데이터의 핵심은 문서가 아니라 entity, relation, event, constraint다.
- 온톨로지는 LLM의 프롬프트 장식이 아니라 기업 AI의 semantic operating system이다.
- 이벤트 온톨로지는 기업 월드모델을 static graph에서 dynamic graph로 바꾼다.
- AI agent가 기업에서 행동하려면 tool 권한보다 먼저 world model과 action validation layer가 필요하다.

---

## 8. 추천 제품 구조

### Phase 1. Ontology-grounded IR

목표:

- 문서 chunk + entity graph + ontology schema 결합
- query를 entity/relation/constraint intent로 분해
- 관련 chunk뿐 아니라 관련 entity path/subgraph를 retrieval

예시 출력:

```text
Answer
Evidence chunks
Relevant entities
Reasoning path
Uncertainty / missing data
```

### Phase 2. Enterprise world model

목표:

- CRM/ERP/BI/문서/로그를 canonical entity graph로 연결
- 현재 상태를 event history 기반으로 계산
- 비용/리스크/운영 상태를 entity에 붙임

예시:

```text
Customer A
  owns Contract B
  uses Product C
  affectedBy Incident D
  has RevenueAtRisk = $320k
```

### Phase 3. Event ontology + simulation

목표:

- 기업 event를 표준 구조로 수집
- event effect를 state/cost/risk model에 반영
- what-if 질문과 impact analysis 지원

예시:

```text
What if supplier X raises price by 8%?
What if service Y is down for 2 hours?
What if this approval is skipped?
```

### Phase 4. Agent action validation

목표:

- agent가 실행하려는 action을 ontology/rule/world state로 검증
- precondition, permission, policy, downstream impact 확인
- audit trace 생성

예시:

```text
Proposed action: grant admin access to user U
Validation:
  User U is contractor
  Contractor cannot receive admin access without SecurityApproval
  SecurityApproval missing
Decision: block action
```

---

## 9. 레퍼런스

1. [Lewis et al., Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks, NeurIPS 2020](https://papers.nips.cc/paper/2020/hash/6b493230205f780e1bc26945df7481e5-Abstract.html)
2. [Pan et al., Unifying Large Language Models and Knowledge Graphs: A Roadmap](https://arxiv.org/abs/2306.08302)
3. [Sequeda, Allemang, Jacob, A Benchmark to Understand the Role of Knowledge Graphs on LLM Accuracy for QA on Enterprise SQL Databases](https://arxiv.org/abs/2311.07509)
4. [Edge et al., From Local to Global: A Graph RAG Approach to Query-Focused Summarization](https://arxiv.org/abs/2404.16130)
5. [Peng et al., Graph Retrieval-Augmented Generation: A Survey](https://arxiv.org/abs/2408.08921)
6. [Sharma et al., OG-RAG: Ontology-Grounded Retrieval-Augmented Generation for Large Language Models](https://arxiv.org/abs/2412.15235)
7. [Li et al., Simple Is Effective: The Roles of Graphs and LLMs in KG-Based RAG](https://arxiv.org/abs/2410.20724)
8. [Singh et al., Data management for developing digital twin ontology model](https://journals.sagepub.com/doi/10.1177/0954405420978117)
9. [Gottschalk and Demidova, EventKG: A Multilingual Event-Centric Temporal Knowledge Graph](https://arxiv.org/abs/1804.04526)
10. [van Hage et al., Design and use of the Simple Event Model](https://www.sciencedirect.com/science/article/pii/S1570826811000199)
11. [W3C, PROV-O: The PROV Ontology](https://www.w3.org/TR/2013/PR-prov-o-20130312/)
12. [W3C, Time Ontology in OWL](https://www.w3.org/TR/owl-time/)
13. [Uschold et al., The Enterprise Ontology](https://www.cambridge.org/core/journals/knowledge-engineering-review/article/abs/the-enterprise-ontology/17080176D5F06DEAEA8DBB2BAA9F8398)

