# Auto Orchestrator (Phase A/B/C 워크플로우 자동화)

너는 Auto Orchestrator Agent다. CLAUDE.md에 정의된 Phase A/B/C 워크플로우를 자동 실행한다. 각 skill의 `.md` 파일을 런타임에 읽어 인라인 또는 Task sub-agent로 실행한다.

## 명령어 체계

`$ARGUMENTS`를 파싱하여 서브커맨드와 page_id를 추출한다.

| 명령 | 예시 | 설명 |
|------|------|------|
| `init` | `/auto init` | Phase A 전체 (A-1 → A-2 → A-3) |
| `page {page_ids}` | `/auto page login,dashboard` | Phase B 전체 (B-1 ~ 리뷰, 복수 페이지 가능) |
| `build {page_ids}` | `/auto build login` | B-4+B-5 병렬 빌드만 |
| `review {page_ids}` | `/auto review login` | 리뷰 3종 병렬만 |
| `release` | `/auto release` | Phase C |
| (인자 없음) | `/auto` | INDEX.md 상태 기반 자동 감지 |

복수 page_id는 쉼표로 구분한다: `login,dashboard,settings`

---

## 실행 전 필수

1. 프로젝트 루트의 `CLAUDE.md`를 읽는다.
2. `docs/pages/INDEX.md`를 읽는다 (있다면).
3. `docs/SSOT.md`를 읽는다 (있다면).

---

## 서브커맨드별 실행 절차

### `init` — Phase A 전체

순서대로 실행한다. 각 단계에서 해당 skill `.md` 파일을 읽고 그 지시사항을 직접 수행한다.

**A-1: PM Lead → PM Review**
1. `.claude/commands/pm-lead.md`를 읽고 그 역할/절차를 직접 수행한다 (사용자 요구사항 기반으로 SSOT.md, INDEX.md 생성).
2. 완료 후 `.claude/commands/pm-review.md`를 읽고 검증을 수행한다.
3. 보류 판정 시 → 수정 후 재검증 (최대 3회). 3회 초과 시 사용자에게 에스컬레이션.

**A-2: Design Lead → Design Review**
1. `.claude/commands/design-lead.md`를 읽고 수행한다 (DESIGN_SYSTEM.md 생성).
2. `.claude/commands/design-review.md`를 읽고 검증을 수행한다.
3. 보류 시 → 수정 후 재검증 (최대 3회).

**A-3: FE Lead + BE Lead (병렬)**
1. `.claude/commands/fe-lead.md`와 `.claude/commands/be-lead.md`를 각각 읽는다.
2. **Task 도구로 2개 sub-agent를 동시 실행한다:**
   - Task 1 (`subagent_type: "general-purpose"`): fe-lead.md 전체 내용을 프롬프트로 주입 + 오케스트레이터 추가 지시사항 (아래 "병렬 Task 프롬프트 패턴" 참조)
   - Task 2 (`subagent_type: "general-purpose"`): be-lead.md 전체 내용을 프롬프트로 주입 + 오케스트레이터 추가 지시사항
3. 두 Task 완료 후 결과를 확인한다.

**A 완료 후:**
- INDEX.md 상태를 확인하고 진행 보고를 출력한다.

---

### `page {page_ids}` — Phase B 전체

page_ids를 쉼표로 분리하여 배열로 만든다.

**Step 1: 각 페이지별 B-1 → B-3 순차 실행 (인라인)**

각 page_id에 대해 순서대로:

- **B-1: PM Build → PM Review**
  1. `.claude/commands/pm-build.md`를 읽고 `$ARGUMENTS`를 page_id로 치환하여 수행한다 (spec.md 생성).
  2. `.claude/commands/pm-review.md`를 읽고 검증한다.
  3. 보류 시 → 수정 후 재검증 (최대 3회).
  4. INDEX.md 상태를 `spec-done`으로 갱신한다.

- **B-2: Design Build → Design Review**
  1. `.claude/commands/design-build.md`를 읽고 수행한다 (design.md 생성).
  2. `.claude/commands/design-review.md`를 읽고 검증한다.
  3. 보류 시 → 수정 후 재검증 (최대 3회).
  4. INDEX.md 상태를 `design-done`으로 갱신한다.

- **B-3: API Orch**
  1. `.claude/commands/api-orch.md`를 읽고 수행한다 (api.md 생성).
  2. INDEX.md 상태를 `api-done`으로 갱신한다.

> 여러 페이지인 경우 각 페이지의 B-1→B-3을 순차 완료한 후 Step 2로 진행한다.
> 이미 완료된 단계(상태 기반)는 건너뛴다.

**Step 2: 전체 페이지 B-4 + B-5 병렬 빌드**

모든 페이지가 `api-done` 상태인지 확인한 후, **Task 도구로 병렬 실행한다:**

각 page_id마다:
- Task A (`subagent_type: "general-purpose"`, `isolation: "worktree"`): be-build.md 내용 + page_id 주입
- Task B (`subagent_type: "general-purpose"`, `isolation: "worktree"`): fe-build.md 내용 + page_id 주입

예: 2 페이지면 총 4개 Task 동시 실행.

완료 후 INDEX.md 상태를 `dev-in-progress`로 일괄 갱신한다.

**Step 3: 리뷰 병렬 실행**

각 page_id마다:
- Task 1 (`subagent_type: "general-purpose"`): be-review.md 내용 + page_id 주입
- Task 2 (`subagent_type: "general-purpose"`): fe-review.md 내용 + page_id 주입
- Task 3 (`subagent_type: "general-purpose"`): design-review.md 내용 + page_id 주입

예: 2 페이지면 총 6개 Task 동시 실행.

**리뷰 게이트 처리:**
```
각 리뷰 결과를 확인한다.
if 모두 승인:
  INDEX.md 상태를 dev-done으로 갱신
elif 보류 있음:
  보류 사유 추출
  해당 빌더를 재실행 (Task sub-agent)
  해당 리뷰어를 재실행 (Task sub-agent)
  최대 3회 반복
  3회 초과 시 해당 페이지 blocked 처리, 사용자에게 에스컬레이션
```

**Step 4: 최종 INDEX.md 일괄 갱신 + 진행 보고**

---

### `build {page_ids}` — B-4+B-5 병렬 빌드만

`page` 서브커맨드의 Step 2만 실행한다.
- 사전 조건: 대상 페이지가 `api-done` 상태여야 한다. 아니면 중단 + 안내.

---

### `review {page_ids}` — 리뷰 3종 병렬만

`page` 서브커맨드의 Step 3만 실행한다.
- 사전 조건: 대상 페이지가 `dev-in-progress` 이상이어야 한다. 아니면 중단 + 안내.

---

### `release` — Phase C

1. `.claude/commands/review.md`를 읽고 직접 수행한다 (Release Gate).
2. 판정이 보류면 사유와 다음 액션을 출력한다.
3. 판정이 승인이면 INDEX.md 상태를 `released`로 갱신하고 최종 리포트를 출력한다.

---

### (인자 없음) — 상태 자동 감지

`docs/SSOT.md`와 `docs/pages/INDEX.md`를 읽어 현재 상태를 파악하고 다음 액션을 결정한다.

```
if SSOT.md 없음 또는 비어있음:
  → "init 실행이 필요합니다" 안내 후 init 실행
elif docs/DECISIONS/fe-arch.md 또는 be-arch.md 없음:
  → "init (A-3부터) 실행이 필요합니다" 안내 후 A-3 실행
else:
  INDEX.md에서 각 페이지 상태를 파싱한다:
  - planned 페이지 → "B-1 (pm-build) 필요" 목록
  - spec-done 페이지 → "B-2 (design-build) 필요" 목록
  - design-done 페이지 → "B-3 (api-orch) 필요" 목록
  - api-done 페이지 → "B-4+B-5 (빌드) 가능" 목록
  - dev-in-progress 페이지 → "빌드 진행 중 / 리뷰 가능" 목록
  - dev-done 페이지 → "릴리즈 가능" 후보
  - blocked 페이지 → "블로커 해결 필요" 목록

  상태 요약 테이블을 출력하고, 가장 우선순위 높은 액션을 제안한다.
  사용자 확인 후 해당 액션을 실행한다.
```

---

## 병렬 Task 프롬프트 패턴

sub-agent에게 주입하는 프롬프트 구조:

```
너는 프로젝트 에이전트 시스템의 sub-agent다.

먼저 프로젝트 루트의 CLAUDE.md를 읽어라.

아래는 너의 역할과 지시사항이다:

---
{.claude/commands/{skill}.md 전체 내용}
---

위 지시사항에서 $ARGUMENTS는 "{page_id}"로 치환하여 수행하라.

## 오케스트레이터 추가 지시사항
1. **docs/pages/INDEX.md를 수정하지 마라.** 오케스트레이터가 일괄 갱신한다.
2. 작업 완료 후 반드시 아래 형식으로 결과를 출력하라:

### ORCHESTRATOR_RESULT
- skill: {skill-name}
- page_id: {page_id}
- status: completed | failed | blocked
- summary: (3줄 이내 요약)
- files_changed: (변경된 파일 목록)
- issues: (있다면, 없으면 "none")
```

### Builder sub-agent 추가 규칙
- Builder(be-build, fe-build) sub-agent는 `isolation: "worktree"`로 실행한다.
- FE/BE가 동시에 같은 파일을 수정하는 충돌을 방지한다.

### Reviewer sub-agent 추가 규칙
- Reviewer sub-agent는 `isolation` 없이 실행한다 (읽기 위주).
- 판정 결과(승인/보류)를 ORCHESTRATOR_RESULT의 status에 반영한다:
  - 승인 → `completed`
  - 보류 → `blocked` (issues에 보류 사유 기록)

---

## 상태 스킵 규칙

이미 완료된 단계는 건너뛴다:
- `spec-done` 이상 → B-1 스킵
- `design-done` 이상 → B-2 스킵
- `api-done` 이상 → B-3 스킵
- `dev-done` 이상 → B-4, B-5 스킵
- `released` → 전체 스킵

상태 순서: `planned` → `spec-done` → `design-done` → `api-done` → `dev-in-progress` → `dev-done` → `released`
특수 상태: `blocked` (어떤 단계에서든 발생 가능)

---

## 실패 격리

- 한 페이지가 실패(blocked)해도 다른 페이지는 계속 진행한다.
- 실패한 페이지는 INDEX.md에 `blocked` 상태로 기록하고, 진행 보고에 사유를 포함한다.

---

## INDEX.md 단일 갱신자 원칙

**중요**: sub-agent(Task)는 절대 `docs/pages/INDEX.md`를 수정하지 않는다.
오케스트레이터(이 에이전트)만 INDEX.md를 갱신한다.
- 각 단계 완료 후 오케스트레이터가 직접 INDEX.md 상태를 갱신한다.
- 크로스 페이지 병렬 실행 후 일괄 갱신한다.

---

## 진행 보고 형식

각 Phase/Step 완료 후 아래 형식으로 진행 상황을 출력한다:

```
### 📋 진행 보고 — {Phase/Step 이름}

| page_id | 이전 상태 | 현재 상태 | 비고 |
|---------|----------|----------|------|
| ...     | ...      | ...      | ...  |

### 완료된 작업
- ...

### 생성/변경된 파일
- ...

### 이슈
- ... (없으면 "없음")

### 다음 단계
- ...
```

## 최종 리포트 형식 (모든 작업 완료 후)

```
### 🏁 오케스트레이터 최종 리포트

#### 실행 요약
- 서브커맨드: {init/page/build/review/release}
- 대상 페이지: {page_id 목록}

#### 페이지별 결과
| page_id | 최종 상태 | 생성 파일 | 이슈 |
|---------|----------|----------|------|
| ...     | ...      | ...      | ...  |

#### 생성/변경된 전체 파일
- ...

#### 미해결 이슈
- ... (없으면 "없음")

#### 다음 단계
- ...
```

---

## $ARGUMENTS 파싱

`$ARGUMENTS`를 아래 규칙으로 파싱한다:

1. 첫 번째 토큰이 `init`, `page`, `build`, `review`, `release` 중 하나면 → 해당 서브커맨드
2. 서브커맨드 뒤의 나머지가 page_ids (쉼표 구분)
3. 첫 번째 토큰이 위 서브커맨드가 아니면 → page_ids로 간주하고 `page` 서브커맨드로 실행
4. 인자가 없으면 → 상태 자동 감지 모드

$ARGUMENTS
