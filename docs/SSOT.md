# SSOT (Single Source of Truth)

> `/pm-lead`가 소유. 프로젝트 수준 개요 + 인덱스. 페이지별 상세는 `docs/pages/{page_id}/spec.md` 참조.

## 프로젝트 개요
<!-- 프로젝트 한 줄 설명 -->

## 핵심 요구사항
<!-- 10~20줄 이내 -->
1. ...

## 페이지 목록
<!-- 상세는 docs/pages/INDEX.md 참조 -->

| page_id | 페이지명 | 상태 |
|---------|---------|------|
| <!-- 예시: login | 로그인 | spec-done --> |

> 전체 페이지 인벤토리, 라우트, 의존성은 [`docs/pages/INDEX.md`](pages/INDEX.md) 참조.

## 사용자 플로우
<!-- 주요 플로우 3~5개 -->
1. ...

## 수용기준 (AC) 요약
<!-- 페이지별 AC 상세는 docs/pages/{page_id}/spec.md 참조 -->

| AC-ID | 설명 | 우선순위 | 페이지 | 상태 |
|-------|------|----------|--------|------|
| AC-xxx-001 | ... | P0 | xxx | 미구현 |

## 기술 스택
<!-- Phase A-3에서 Lead가 결정. 상세는 DECISIONS/fe-arch.md, DECISIONS/be-arch.md 참조 -->

| 영역 | 기술 | 비고 |
|------|------|------|
| FE Framework | <!-- 예: Next.js 14 --> | |
| BE Framework | <!-- 예: NestJS --> | |
| Database | <!-- 예: PostgreSQL --> | |
| Infra | <!-- 예: Docker + Nginx --> | |
| 프로젝트 구조 | <!-- 모노레포 / 폴리레포 --> | |

## 비기능 요구사항
- 성능: ...
- 보안: ...
- 가용성: ...

## 디자인 시스템 요약
<!-- 상세는 DESIGN_SYSTEM.md 참조 -->
- 토큰 체계: 시맨틱 컬러, Neutral 스케일, Surface 계층, 타이포, 스페이싱(8pt), 그림자, 보더, 모션
- 접근성: WCAG 2.1 AA (대비비 4.5:1 / 3:1)
- 다크모드: CSS 변수 기반 테마 전환

## API 표준 요약
<!-- 상세는 API_STANDARDS.md 참조 -->
- 인증: Bearer Token
- 에러 포맷: `{ error: { code, message, details } }`
- 페이지네이션: offset 기반 또는 cursor 기반

## Decision Log 요약
<!-- 상세는 docs/DECISIONS/*.md 참조 -->

| # | 결정 | 파일 | 날짜 |
|---|------|------|------|
