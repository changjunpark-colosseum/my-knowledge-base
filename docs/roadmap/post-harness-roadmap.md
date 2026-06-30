# Post-Harness Roadmap

이 문서는 **“하니스 이후에 무엇을 만들어야 하는가”**에 대한 로드맵이다.  
전제는 간단하다:

- 하니스 자체는 이제 점점 **기반 인프라**가 되고 있고
- 다음 경쟁력은 **하니스 위에서 실제로 굴러가는 시스템**에서 나온다

즉, 다음 단계는 “좋은 harness를 만들기”보다  
**harness 위에서 검증 가능하고 지속적으로 돌아가는 AI 작업 운영체계**를 만드는 것이다.

## 한 줄 요약

> Harness → Runtime OS → Closed-loop execution system → Domain-specific autonomous production

## 왜 다음 단계가 필요한가

하니스만으로는 이제 차별화가 약해질 수 있다.

- 멀티에이전트 실행
- 상태 저장
- 팀 라우팅
- prompt/skill orchestration
- tmux 기반 worker 운영

이런 요소는 앞으로 점점 **기본기**가 된다.  
그러면 진짜 차이는 다음 질문에서 생긴다.

- 어떤 일을 자동으로 계속 굴릴 수 있는가
- 실패를 어떻게 복구하는가
- 검증을 어떻게 강제하는가
- 어떤 도메인에서 더 잘 작동하는가
- 운영 데이터가 축적되는가

## Phase 1. Harness standardization

목표: 지금 가진 harness를 **비교 가능하고 재사용 가능하게 표준화**한다.

### 해야 할 일
- 툴별 공통 문서 포맷 정리
- runtime / workflow / state / verification 축으로 비교표 만들기
- tool-selection matrix 작성
- 상황별 추천 규칙 정의

### 산출물
- `docs/tool-selection-matrix.md`
- `docs/runtime-comparison.md`
- `docs/evaluation-criteria.md`

### 완료 기준
- “어떤 상황에 어떤 툴을 쓸지”를 문서 하나로 설명 가능
- 새 툴이 들어와도 같은 기준으로 비교 가능

## Phase 2. Closed-loop execution

목표: 단순 실행이 아니라 **계획 → 실행 → 검증 → 재시도 → 보고**의 폐루프를 만든다.

### 핵심 아이디어
좋은 시스템은 agent를 한 번 돌리고 끝나지 않는다.  
반드시 다음을 포함해야 한다.

1. task decomposition
2. execution
3. verification
4. retry / fallback
5. artifact generation
6. final decision

### 필요한 구성
- acceptance criteria
- verification gate
- retry policy
- failure classification
- rollback / recovery 전략
- trace/log 저장

### 산출물
- 실행 파이프라인 문서
- verification contract
- failure taxonomy
- benchmark 시나리오

## Phase 3. Persistent worker systems

목표: 세션성 agent를 넘어서 **장기 실행 worker fleet**로 이동한다.

### 예시
- backlog triage worker
- PR review worker
- docs drift detector
- release readiness worker
- refactor suggestion worker
- regression reproduction worker

### 왜 중요한가
이 단계부터는 “도구”가 아니라  
**AI software organization** 에 가까워진다.

### 요구 조건
- worker identity
- durable state
- queue / mailbox
- owner / approval model
- health check / heartbeat
- pause / resume / shutdown

## Phase 4. Domain-specialized harness packs

목표: 범용 agent보다 **좁고 강한 domain pack**을 만든다.

### 예시
- frontend harness
- repo maintenance harness
- refactor harness
- incident/oncall harness
- documentation harness
- design-system harness

### 이유
범용 모델은 평준화되기 쉽다.  
하지만 도메인별로:

- workflow
- verification
- artifact
- risk model

이 달라지기 때문에, 여기서 실질적인 차별화가 나온다.

## Phase 5. Evaluation moat

목표: “잘 돌아가는 느낌”이 아니라 **측정 가능한 운영 체계**를 만든다.

### 반드시 쌓아야 할 것
- 성공/실패 로그
- task 유형별 성능
- agent 조합별 품질
- retry율
- verification 실패 패턴
- human intervention 빈도
- cost / latency / reliability

### 핵심
다음 경쟁력은 harness 자체보다  
**evaluation + feedback loop + 운영 데이터**에 있다.

## 추천 우선순위

실제로는 아래 순서가 가장 현실적이다.

### 1순위: 비교 체계 만들기
- tool-selection matrix
- runtime comparison
- evaluation criteria

### 2순위: 폐루프 구축
- plan → execute → verify → retry 자동화
- acceptance criteria / verification contract 정의

### 3순위: 지속 worker 만들기
- backlog / PR / docs / release worker

### 4순위: 도메인 팩 만들기
- frontend
- refactor
- docs
- maintenance

### 5순위: 운영 데이터 축적
- benchmark
- scorecard
- failure log
- tuning history

## 추천 문서 트리

post-harness 단계에서 추천하는 문서 구조는 이렇다.

- `docs/post-harness-roadmap.md`
- `docs/tool-selection-matrix.md`
- `docs/runtime-comparison.md`
- `docs/evaluation-criteria.md`
- `docs/closed-loop-execution.md`
- `docs/persistent-workers.md`
- `docs/domain-packs.md`
- `docs/experiment-log.md`

## 이 workspace 기준 다음 액션

지금 이 workspace에서 바로 이어가기 좋은 순서는:

1. `tool-selection-matrix.md`
2. `runtime-comparison.md`
3. `evaluation-criteria.md`
4. `closed-loop-execution.md`

그 다음에:

5. `persistent-workers.md`
6. `experiment-log.md`

## 결론

하니스가 끝났다는 말은  
하니스가 쓸모없다는 뜻이 아니라,
**이제 하니스는 시작점이라는 뜻**에 가깝다.

다음 단계는:

- 더 많은 agent를 붙이는 것보다
- 더 많은 prompt를 추가하는 것보다
- **검증 가능하고 지속적으로 운영되는 자율 생산 시스템**을 만드는 것

이다.

## 관련 문서

- [index.md](./index.md)
- [harness-engineering.md](./harness-engineering.md)
- [oh-my-codex.md](./oh-my-codex.md)
