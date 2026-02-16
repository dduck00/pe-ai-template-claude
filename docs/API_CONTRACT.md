# API Contract

## Base URL
- Production: `https://your-domain.com/api`
- Development: `http://localhost:8080/api`

## 인증
<!-- JWT, Session, API Key 등 -->

## 공통 에러 포맷
```json
{
  "error": "ERROR_CODE",
  "message": "Human-readable message"
}
```

## Endpoints

### 인증 API

#### POST /api/auth/login
- Request: `{ "email": "string", "password": "string" }`
- Response 200: `{ "token": "string" }`
- Error 401: `{ "error": "INVALID_CREDENTIALS" }`

<!-- 엔드포인트 추가 -->
