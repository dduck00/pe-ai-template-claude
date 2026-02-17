# Project Agent System v2

## Agent 구조 (14개)

```
Agents
├─ PM Group
│  ├─ PM Lead (/pm-lead)              - 요구사항 → 기능/페이지 정의, SSOT + INDEX 유지
│  ├─ PM Builder (/pm-build)          - 페이지별 상세 스펙 작성
│  └─ PM Reviewer (/pm-review)        - 크로스페이지 정합성 검증
├─ Design Group
│  ├─ Design Lead (/design-lead)      - 프로젝트 디자인 가이드라인 수립
│  ├─ Design Builder (/design-build)  - 페이지별 상세 디자인 적용
│  └─ Design Reviewer (/design-review)- 디자인 충실도 검증
├─ Orchestrator
│  └─ API Orchestrator (/api-orch)    - 페이지별 API 정의 (req/res)
├─ FE Group
│  ├─ FE Lead (/fe-lead)             - FE 아키텍처 가드레일
│  ├─ FE Builder (/fe-build)         - 구현 + 테스트
│  └─ FE Reviewer (/fe-review)       - FE 코드 리뷰 + 테스트 검증
├─ BE Group
│  ├─ BE Lead (/be-lead)             - BE 아키텍처 + 도메인 정의
│  ├─ BE Builder (/be-build)         - 구현 + 테스트
│  └─ BE Reviewer (/be-review)       - BE 코드 리뷰 + 테스트 검증
└─ Release
   └─ Release Gate (/review)         - 최종 릴리즈 판정 (통합)
```

## 문서 권위 체계 (2-Tier)

### Tier 1: 글로벌 문서 (프로젝트 수준)

| 파일 | 소유 | 역할 |
|------|------|------|
| `docs/SSOT.md` | /pm-lead | 프로젝트 개요 + 인덱스 (1~2페이지) |
| `docs/pages/INDEX.md` | /pm-lead | 페이지 인벤토리, 라우트, 상태, 의존성 |
| `docs/DESIGN_SYSTEM.md` | /design-lead | 디자인 시스템 토큰 + 방법론 + 체크리스트 |
| `docs/API_STANDARDS.md` | /api-orch + /be-lead | API 공통 규칙 (인증, 페이징, 에러, 네이밍) |
| `docs/DECISIONS/*.md` | 공유 (ADR 방식) | 아키텍처/정책 결정 (append-only) |
| `docs/DEPLOY.md` | /review | 환경변수, 인프라, 배포 런북 |
| `docs/TEST_PLAN.md` | /review | 테스트 현황/갭/AC 추적성 |

### Tier 2: 페이지별 문서 (폴더 분리)

```
docs/pages/
├─ INDEX.md
├─ {page_id}/
│  ├─ spec.md      ← PM Builder 작성 (목적, 유저스토리, AC, 상태, 엣지케이스)
│  ├─ design.md    ← Design Builder 작성 (UI/UX, 컴포넌트, 인터랙션, 접근성)
│  ├─ api.md       ← API Orchestrator 작성 (엔드포인트, req/res, 에러)
│  └─ tests.md     ← FE/BE Reviewer 갱신 (테스트 케이스, 커버리지)
```

### 충돌 우선순위

- API 공통 규칙: `API_STANDARDS.md`
- 페이지별 API: `docs/pages/{page_id}/api.md`
- 아키텍처/정책: `docs/DECISIONS/*.md`
- 제품 범위/AC: `SSOT.md` → `docs/pages/{page_id}/spec.md`
- 디자인 토큰/스펙: `DESIGN_SYSTEM.md` → `docs/pages/{page_id}/design.md`
- 인프라/배포: `DEPLOY.md`
- 문서 간 충돌 발견 시 `/pm-lead`가 판단하고 `docs/DECISIONS/`에 기록한다.

## 호출 플로우

### Phase A: 프로젝트 초기 (1회)

| 단계 | 에이전트 | 입력 (읽기) | 산출물 (쓰기) | Gate |
|------|----------|-------------|---------------|------|
| A-1 | `/pm-lead` | 요구사항 | SSOT.md, INDEX.md | /pm-review |
| A-2 | `/design-lead` | SSOT.md | DESIGN_SYSTEM.md | /design-review |
| A-3 | `/fe-lead` | SSOT, DESIGN_SYSTEM | DECISIONS/fe-arch.md | - |
| A-3 | `/be-lead` | SSOT, API_STANDARDS | DECISIONS/be-arch.md | - |

> **기술 스택 결정**: FE/BE 기술 스택(프레임워크, 언어, DB 등)은 A-3에서 각 Lead가 `DECISIONS/fe-arch.md`, `DECISIONS/be-arch.md`에 기록한다. 프로젝트 구조(모노레포/폴리레포)도 이 단계에서 결정한다.

### Phase B: 페이지별 반복 (페이지마다)

| 단계 | 에이전트 | 입력 (읽기) | 산출물 (쓰기) | Gate |
|------|----------|-------------|---------------|------|
| B-1 | `/pm-build {page_id}` | SSOT, INDEX | pages/{page_id}/spec.md | /pm-review |
| B-2 | `/design-build {page_id}` | spec.md, DESIGN_SYSTEM | pages/{page_id}/design.md | /design-review |
| B-3 | `/api-orch {page_id}` | spec.md, design.md, API_STANDARDS | pages/{page_id}/api.md | /be-lead 승인 |
| B-4 | `/be-build {page_id}` | api.md, be-arch.md | BE 코드 + 테스트 | /be-review |
| B-5 | `/fe-build {page_id}` | spec+design+api.md, fe-arch.md | FE 코드 + 테스트 | /fe-review, /design-review |

#### 병렬 작업 규칙

- **B-4 + B-5 병렬 가능**: 동일 page_id에서 B-3(api.md 확정) 이후 BE/FE Builder를 동시에 실행할 수 있다.
- **서로 다른 page_id 병렬 가능**: page_id간 의존성이 없으면 각각의 Phase B를 동시에 진행할 수 있다. 단, INDEX.md 갱신 시 충돌에 주의한다.
- **병렬 불가 구간**: B-1 → B-2 → B-3은 순차 실행 (각 산출물이 다음 단계의 입력).

### Phase C: 릴리즈 (마일스톤)

| 단계 | 에이전트 | 입력 (읽기) | 산출물 (쓰기) | Gate |
|------|----------|-------------|---------------|------|
| C-1 | `/review` | INDEX, 페이지 docs, 테스트 결과 | 릴리즈 판정, TEST_PLAN, DEPLOY.md | 최종 |

## 리뷰 피드백 루프

- Reviewer가 **"보류"** 판정 시, 보류 사유를 고정 포맷으로 출력한다.
- Builder는 보류 사유를 반영하여 수정 후, **동일 Reviewer를 재호출**한다.
- 재리뷰 범위는 **수정분 diff + 보류 사유 항목**으로 한정한다 (전체 재검증 아님).
- 최대 **3회 iteration**까지 허용한다. 3회 초과 시 해당 Lead에 에스컬레이션한다.
- 보류 사유는 Reviewer 출력에 포함되며, 별도 파일 기록은 불요. 단, 반복 보류 시 `docs/DECISIONS/`에 원인 분석 기록을 권장한다.

## 예외/롤백 규칙

- API 계약 변경: `/pm-lead`가 판단 → `/api-orch`가 api.md 갱신 → Lead 재검토 → Builder 재개
- 보안 고위험 변경: 즉시 `/review` 실행, 해결 전 릴리즈 동결
- Lead 간 결정 충돌: `/pm-lead`가 24시간 내 단일안 확정, `docs/DECISIONS/`에 근거 기록
- 24시간 이상 미해결 블로커: 신규 구현 중단, 스펙 정리 작업만 진행
- 릴리즈 후 P1 결함: 직전 안정 버전으로 롤백, 원인/재발방지를 `docs/DECISIONS/`와 `docs/TEST_PLAN.md`에 추가

---

## 공통 원칙

### SSOT 운영 원칙

- 모든 에이전트는 문서 정본을 기준으로 판단한다 (대화가 아닌 파일).
- Tier 1 문서는 프로젝트 수준 개요/인덱스. Tier 2 문서는 페이지별 상세.
- 새 세션 컨텍스트 복원: SSOT(1) + INDEX(1) + 해당 페이지 폴더(2~3파일) = 4~5파일.

### SSOT ↔ INDEX 동기화 규칙

- SSOT.md의 페이지 요약은 INDEX.md에서 파생된다.
- `/pm-lead`가 INDEX.md를 먼저 갱신한 후, SSOT.md의 페이지 요약을 동기화한다.
- INDEX.md가 페이지 목록의 정본(source of truth)이다.

### 고정 포맷 출력

- Lead/Reviewer 출력 템플릿(장문 금지, 각 항목 1~3줄):
- `결정`:
- `근거`:
- `대안`:
- `리스크`:
- `검증`:

### 변경분 중심 리뷰

- 검증/리뷰 범위는 "이번 변경분(diff) + 핵심 파일 + 문서 정본 변경점"으로 제한한다.
- 리뷰 코멘트는 가능한 한 AC ID 또는 API 계약 항목과 연결한다.

### Build 에이전트 $ARGUMENTS 규칙

- `/pm-build`, `/design-build`, `/api-orch`, `/fe-build`, `/be-build`는 `$ARGUMENTS`로 page_id를 받는다.
- page_id가 없으면 `docs/pages/INDEX.md`를 보여주고 중단한다.
- Reviewer(`/fe-review`, `/be-review`)도 `$ARGUMENTS`로 page_id를 받을 수 있다 (optional). 없으면 최근 변경 diff 기반으로 리뷰.

### 페이지 문서 템플릿

- Build 에이전트는 페이지 문서 최초 생성 시 아래 템플릿을 참조한다:
  - `docs/pages/TEMPLATE_spec.md` — PM Builder용 스펙 템플릿
  - `docs/pages/TEMPLATE_design.md` — Design Builder용 디자인 템플릿
  - `docs/pages/TEMPLATE_api.md` — API Orchestrator용 API 템플릿
  - `docs/pages/TEMPLATE_tests.md` — FE/BE Reviewer용 테스트 현황 템플릿
- ADR 작성 시 `docs/DECISIONS/TEMPLATE.md`를 참조한다.

### Lead 결정 영속화

- Lead 에이전트의 결정은 `docs/DECISIONS/`에 ADR 형태로 파일에 저장한다.
- ADR 추가/변경 시 `docs/DECISIONS/INDEX.md`도 함께 갱신한다.
- 세션이 종료되어도 결정이 유지된다.

---

## 산출물 파일

### Tier 1 (글로벌)

| 파일 | 소유 | 갱신 트리거 |
|------|------|-------------|
| `docs/SSOT.md` | /pm-lead | 요구사항/AC/우선순위 변경 시 |
| `docs/pages/INDEX.md` | /pm-lead | 페이지 추가/상태 변경 시 |
| `docs/DESIGN_SYSTEM.md` | /design-lead | 디자인 토큰/방법론 변경 시 |
| `docs/API_STANDARDS.md` | /api-orch + /be-lead | API 공통 규칙 변경 시 |
| `docs/DECISIONS/*.md` | 공유 | 아키텍처/정책 결정 시 |
| `docs/DEPLOY.md` | /review | 환경변수/인프라/배포 변경 시 |
| `docs/TEST_PLAN.md` | /review | 릴리즈 Gate 수행 시 |

### Tier 2 (페이지별)

| 파일 | 소유 | 갱신 트리거 |
|------|------|-------------|
| `docs/pages/{page_id}/spec.md` | /pm-build | 페이지 스펙 작성/변경 시 |
| `docs/pages/{page_id}/design.md` | /design-build | 페이지 디자인 작성/변경 시 |
| `docs/pages/{page_id}/api.md` | /api-orch | 페이지 API 정의/변경 시 |
| `docs/pages/{page_id}/tests.md` | /fe-review, /be-review | 코드 리뷰 시 테스트 현황 기록 |

---

## 표준 체크리스트

### FE 최소 체크

- [ ] 권한 가드 / 라우팅 보호 / 토큰 저장 원칙
- [ ] 입력 처리 (XSS 방어) / 에러·로딩·빈 상태 UX
- [ ] 주요 플로우 단위 테스트 + AC 매핑
- [ ] 주요 사용자 시나리오 E2E 테스트 (FE Builder 책임)

### BE 최소 체크

- [ ] 인증·인가·권한 스코프 / 입력 검증 + DTO 경계
- [ ] 트랜잭션·인덱스·쿼리 / 에러코드 표준화
- [ ] 단위 + 핵심 통합 테스트 + AC 매핑
- [ ] API 엔드포인트 통합 테스트 (BE Builder 책임)

### Design 최소 체크

- [ ] 시맨틱 토큰 사용 (하드코딩 hex/rgb 없음)
- [ ] 8pt 그리드 스페이싱 준수
- [ ] 컬러 대비비 WCAG AA (일반 4.5:1, 대형 3:1)
- [ ] `docs/DESIGN_SYSTEM.md` 최신 상태

### Infra 최소 체크

- [ ] 환경변수 소스 일치 + 포맷 유효성
- [ ] Docker TZ + Nginx/Proxy 설정
- [ ] `docs/DEPLOY.md` 최신 상태

## AC-테스트 추적성 규칙

- 모든 수용기준은 `AC-{page_id}-NNN` ID를 유지한다.
- `docs/TEST_PLAN.md`에 `AC-ID | Test Case | Type(Unit/Integration/E2E) | Status` 테이블을 유지한다.
- `docs/pages/{page_id}/tests.md`에 페이지별 테스트 현황을 기록한다.
- Reviewer는 결함/갭을 AC-ID 기준으로 기록한다.

### 테스트 유형별 책임

| 유형 | 담당 | 범위 |
|------|------|------|
| 단위 테스트 | FE/BE Builder 각각 | 개별 함수/컴포넌트 로직 |
| 통합 테스트 | BE Builder | API 엔드포인트 요청-응답 검증 |
| E2E 테스트 | FE Builder | 주요 사용자 시나리오 (로그인→핵심기능) |
| 테스트 갭 분석 | /review (Phase C) | TEST_PLAN.md에 미커버 AC 식별 |

---

## 확장 가이드

### 대규모 프로젝트 (30+ 페이지)

- INDEX.md가 비대해지면 페이지를 모듈/도메인별로 그룹핑하고, INDEX.md에 그룹 섹션을 나눈다.
- SSOT.md "1~2페이지" 제약이 부족하면 모듈별 요약으로 전환한다.
- `docs/DECISIONS/INDEX.md`에서 현행(`accepted`) ADR만 필터링하여 관리한다.

### 다중 사용자 동시 작업

- 이 템플릿은 1인 Claude Code 기반 순차 작업에 최적화되어 있다.
- 여러 개발자가 동시 작업 시: 각자 다른 page_id를 담당하고, SSOT/INDEX 수정은 한 명이 담당하여 충돌을 방지한다.
- Git branching: page_id별 feature branch 사용을 권장한다 (예: `feature/{page_id}`).
