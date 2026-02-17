# API Standards (정본)

> `/api-orch` + `/be-lead`가 공동 소유. 모든 페이지별 API 정의(`docs/pages/{page_id}/api.md`)는 이 표준을 따른다.

## 인증 & 인가

- 인증 방식: Bearer Token (Authorization 헤더)
- 토큰 포맷: JWT 또는 프로젝트 정의 방식
- 권한 모델: RBAC 또는 프로젝트 정의 방식
- 민감 엔드포인트: 추가 검증 필수 (2FA, re-auth 등)

## URL 네이밍

- 기본: `/{version}/{resource}` (복수형, kebab-case)
- 버전: URL prefix (`/v1/`) 또는 헤더 기반
- 리소스 중첩: 최대 2단계 (`/users/{id}/posts`)
- 필터/검색: query parameter 사용 (`?status=active&sort=created_at`)

## 요청/응답 포맷

### 성공 응답

```json
{
  "data": { ... },
  "meta": { "page": 1, "total": 100 }
}
```

- 단일 리소스: `data`에 객체
- 목록: `data`에 배열 + `meta`에 페이지네이션

### 에러 응답

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Human-readable message",
    "details": [ ... ]
  }
}
```

## HTTP 상태 코드

| 코드 | 용도 |
|------|------|
| 200 | 성공 (GET, PUT, PATCH) |
| 201 | 생성 성공 (POST) |
| 204 | 삭제 성공 (DELETE) |
| 400 | 입력 검증 실패 |
| 401 | 인증 실패 |
| 403 | 권한 부족 |
| 404 | 리소스 없음 |
| 409 | 충돌 (중복 등) |
| 422 | 비즈니스 로직 검증 실패 |
| 500 | 서버 내부 오류 |

## 페이지네이션

- 방식: offset 기반 (`?page=1&size=20`) 또는 cursor 기반
- 기본 size: 20, 최대: 100
- 응답 meta: `page`, `size`, `total`, `totalPages`

## 공통 헤더

| 헤더 | 방향 | 용도 |
|------|------|------|
| Authorization | req | Bearer 토큰 |
| Content-Type | req/res | `application/json` |
| Accept-Language | req | 다국어 (선택) |
| X-Request-Id | req/res | 요청 추적 |

## 입력 검증

- 모든 입력은 서버에서 재검증 (클라이언트 검증에 의존하지 않음)
- DTO 경계에서 타입/범위/필수값 검증
- 검증 실패: 400 + details 배열에 필드별 에러

## 버전 관리

- 하위 호환 변경: 필드 추가, 선택적 파라미터 추가
- 하위 비호환 변경: 새 버전 엔드포인트 생성, 이전 버전 deprecation 공지
