# Project Agent System

## Agent 구조

```
Agents
├─ Global
│  ├─ PM & Spec Agent (/pm)        - Spec Owner + Conflict Resolver
│  ├─ Deploy Agent (/deploy)       - Infra/Env Auditor + Deployment Gate
│  └─ Release Reviewer (/review)   - PR/Release Gate Owner (인프라 포함)
├─ FE Group
│  ├─ FE Lead (/fe-lead)           - Design System + FE Architecture + FE Security
│  └─ FE Builder (/fe-build)       - Implementation + Unit Tests
└─ BE Group
   ├─ BE Lead (/be-lead)           - DB + API Architecture + BE Security
   └─ BE Builder (/be-build)       - Implementation + Unit/Integration Tests
```

## 문서 권위 체계

- `docs/SSOT.md`: 제품 요구사항/AC/우선순위의 요약 본문(인덱스 포함)
- `docs/API_CONTRACT.md`: API 인터페이스의 정본
- `docs/DECISIONS.md`: 의사결정의 정본
- `docs/TEST_PLAN.md`: 테스트 현황/갭/AC 추적성의 정본
- `docs/DEPLOY.md`: 환경변수 계약/인프라 인벤토리/배포 런북의 정본
- 충돌 우선순위: API 내용은 `API_CONTRACT.md`, 아키텍처/정책은 `DECISIONS.md`, 제품 범위/AC는 `SSOT.md`, 인프라/배포는 `DEPLOY.md`를 따른다.
- 문서 간 충돌 발견 시 `/pm`이 24시간 내 최종 판단하고 `docs/DECISIONS.md`에 기록한다.

## 호출 플로우 (기본)

| 단계 | 입력 | 산출물 | Exit Criteria | 실패/충돌 시 |
|------|------|--------|---------------|---------------|
| 1. `/pm` | 요구사항, 기존 문서 | SSOT 갱신, 필요 시 API/Decision 변경 요청 | AC ID 부여, 오픈 이슈/가정 명시 | 정보 부족 시 블로킹 이슈 등록 후 질문 |
| 2. `/fe-lead` `/be-lead` | SSOT, API_CONTRACT, DECISIONS | 결정문(고정 포맷), 가드레일, 리스크 | FE/BE 결정이 상호 모순 없음 | 모순 시 `/pm`으로 에스컬레이션 |
| 3. `/fe-build` `/be-build` | Lead 결정문, 확정 문서 | 코드, 테스트, 변경 요약 | 핵심 테스트 통과, AC 매핑 업데이트 | 계약 변경 감지 시 즉시 중단 후 1단계 재진입 |
| 3.5 `/deploy` | 코드, 인프라 파일, DEPLOY.md | env 감사, 인프라 리뷰, 체크리스트 | 필수 env non-empty + 포맷 유효 + 소스 일치 | 즉시 수정 후 재검증 |
| 4. `/review` (PR Gate) | PR diff, 핵심 파일, 문서 변경점 | 결함 목록, 테스트 갭, 게이트 판정 (코드+인프라) | Open High 0건 | High 발생 시 merge 차단 |
| 5. `/review` (Release Gate) | 릴리즈 후보, TEST_PLAN | 최종 릴리즈 판정 | Open High 0건, Medium 승인/수용 | 실패 시 릴리즈 보류 + 수정 계획 |

## 예외/롤백 규칙

- API 계약 변경: `/pm`이 `docs/API_CONTRACT.md` 먼저 갱신하고 Lead 재검토 후 Builder 재개
- 보안 고위험 변경: 즉시 PR Gate 실행, 해결 전 릴리즈 동결
- Lead 간 결정 충돌: `/pm`이 24시간 내 단일안 확정, `docs/DECISIONS.md`에 근거 기록
- 24시간 이상 미해결 블로커: 신규 구현 중단, 스펙 정리 작업만 진행
- 릴리즈 후 P1 결함: 직전 안정 버전으로 롤백, 원인/재발방지 항목을 `docs/DECISIONS.md`와 `docs/TEST_PLAN.md`에 추가

---

## 공통 원칙

### SSOT 운영 원칙

- 모든 에이전트는 "전체 대화"가 아니라 문서 정본(`SSOT`, `API_CONTRACT`, `DECISIONS`, `TEST_PLAN`)을 기준으로 판단한다.
- `docs/SSOT.md`는 요약/인덱스 역할을 수행하며, 상세 계약/결정은 각 정본 문서를 참조한다.

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

---

## 산출물 파일

| 파일 | 소유 | 위치 | 갱신 트리거 |
|------|------|------|-------------|
| SSOT.md | PM & Spec | `docs/SSOT.md` | 요구사항/AC/우선순위 변경 시 |
| API_CONTRACT.md | PM & Spec | `docs/API_CONTRACT.md` | 엔드포인트/인증/에러 포맷 변경 시 |
| DECISIONS.md | 공유 | `docs/DECISIONS.md` | 아키텍처/정책 결정 시 (24시간 내 기록) |
| TEST_PLAN.md | Reviewer | `docs/TEST_PLAN.md` | PR/Release Gate 수행 시 |
| DEPLOY.md | Deploy | `docs/DEPLOY.md` | 환경변수/인프라/배포 설정 변경 시 |

---

## 표준 체크리스트

### FE 최소 체크

- [ ] 권한 가드 / 라우팅 보호 / 토큰 저장 원칙
- [ ] 입력 처리 (XSS 방어) / 에러·로딩·빈 상태 UX
- [ ] 주요 플로우 단위 테스트 + AC 매핑

### BE 최소 체크

- [ ] 인증·인가·권한 스코프 / 입력 검증 + DTO 경계
- [ ] 트랜잭션·인덱스·쿼리 / 에러코드 표준화
- [ ] 단위 + 핵심 통합 테스트 + AC 매핑

### Infra 최소 체크

- [ ] 환경변수 소스 일치 + 포맷 유효성
- [ ] Docker TZ + Nginx/Proxy 설정
- [ ] `docs/DEPLOY.md` 최신 상태

## AC-테스트 추적성 규칙

- 모든 수용기준은 `AC-xxx` ID를 유지한다.
- `docs/TEST_PLAN.md`에 `AC-ID | Test Case | Type(Unit/Integration/E2E) | Status` 테이블을 유지한다.
- Reviewer는 결함/갭을 AC-ID 기준으로 기록한다.
