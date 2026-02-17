# {page_id} — API 정의

> `/api-orch`가 작성. `docs/API_STANDARDS.md` 규칙 준수.

## 엔드포인트 목록

| Method | Path | 설명 | 인증 |
|--------|------|------|------|
| <!-- 예시: POST | /v1/auth/login | 로그인 | 불필요 --> |

## 상세 정의

### `METHOD /path`

**설명**: ...

**Request**

| 위치 | 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|------|
| body | email | string | Y | 이메일 |

```json
{
  "email": "user@example.com"
}
```

**Response 200**

```json
{
  "data": { ... }
}
```

**에러 케이스**

| 상태 코드 | error.code | 조건 |
|-----------|------------|------|
| 400 | VALIDATION_ERROR | 입력 형식 오류 |
| 401 | UNAUTHORIZED | 인증 실패 |

---

## 공통 사항

- 인증: `docs/API_STANDARDS.md` 참조
- 에러 포맷: `docs/API_STANDARDS.md` 참조
- 페이지네이션: ... (해당 시)

## AC 매핑

| AC-ID | 관련 엔드포인트 |
|-------|----------------|
| AC-XXX-001 | `POST /v1/...` |

## 변경 이력

| 날짜 | 변경 내용 | 작성자 |
|------|-----------|--------|
