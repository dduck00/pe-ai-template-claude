# Page Index

> `/pm-lead`가 소유. 프로젝트의 모든 페이지 인벤토리.

## 페이지 목록

| page_id | 페이지명 | 라우트 | 상태 | 의존 페이지 | 담당 | 비고 |
|---------|---------|--------|------|------------|------|------|
| <!-- 예시: login | 로그인 | /login | spec-done | - | - | - --> |

## 상태 정의

| 상태 | 설명 |
|------|------|
| `planned` | 페이지 식별됨, 스펙 미작성 |
| `spec-draft` | spec.md 작성 중 |
| `spec-done` | spec.md 완료, PM Review 통과 |
| `design-done` | design.md 완료, Design Review 통과 |
| `api-done` | api.md 완료, BE Lead 승인 |
| `dev-in-progress` | FE/BE 구현 중 |
| `dev-done` | FE/BE 구현 완료, 코드 리뷰 통과 |
| `released` | 릴리즈 완료 |
| `blocked` | 블로커로 인해 진행 중단 (사유를 비고에 기록) |
| `deprecated` | 페이지 폐기 (사유를 비고에 기록) |

## 페이지 의존성 맵

```
<!-- 예시:
login ──→ dashboard
settings ──→ (독립)
-->
```

## 갱신 규칙

- `/pm-lead`가 새 페이지 식별 시 행 추가
- `/pm-build`가 spec 완료 시 상태 갱신
- 각 단계 완료 시 해당 에이전트가 상태 갱신
