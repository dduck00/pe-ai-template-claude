# Design System (정본)

> `/design-lead`가 소유. 디자인 토큰/컴포넌트 스펙/방법론의 단일 진실 원천.
> 페이지별 디자인 상세는 `docs/pages/{page_id}/design.md` 참조.

---

## 디자인 원칙

1. **정보 위계**: 크기 → 굵기 → 색상 순으로 구분 (3단계 이내)
2. **밀도**: 충분한 여백, 요소 간 breathing room 확보
3. **일관성**: 동일 패턴의 동일 시각 처리
4. **절제**: 장식 최소화, 콘텐츠 중심, 그림자/보더 남용 금지
5. **모던 트렌드**: 미니멀, 넉넉한 여백, 부드러운 색상 전환, 미세한 그라데이션

---

## 시맨틱 컬러

| 토큰 | Light | Dark | 용도 | 대비비 |
|------|-------|------|------|--------|
| color-primary | — | — | 주요 액션, CTA | ≥4.5:1 |
| color-secondary | — | — | 보조 액션 | ≥4.5:1 |
| color-accent | — | — | 강조, 배지 | ≥3:1 |
| color-success | — | — | 성공 상태 | ≥4.5:1 |
| color-warning | — | — | 경고 상태 | ≥3:1 |
| color-error | — | — | 에러 상태 | ≥4.5:1 |
| color-info | — | — | 정보성 안내 | ≥4.5:1 |

## Neutral 스케일

| 토큰 | Light | Dark | 용도 |
|------|-------|------|------|
| neutral-50 | — | — | 배경 (가장 밝음) |
| neutral-100 | — | — | 카드 배경 |
| neutral-200 | — | — | 보더, 구분선 |
| neutral-300 | — | — | 비활성 보더 |
| neutral-400 | — | — | 플레이스홀더 |
| neutral-500 | — | — | 보조 텍스트 |
| neutral-600 | — | — | 아이콘 |
| neutral-700 | — | — | 본문 텍스트 |
| neutral-800 | — | — | 제목 텍스트 |
| neutral-900 | — | — | 가장 진한 텍스트 |

## Surface 계층

| 토큰 | Light | Dark | 용도 |
|------|-------|------|------|
| surface-background | — | — | 페이지 배경 |
| surface-0 | — | — | 카드/패널 기본 |
| surface-1 | — | — | 호버/선택 배경 |
| surface-2 | — | — | 모달/드롭다운 |
| surface-3 | — | — | 최상위 오버레이 |

---

## 타이포그래피

| 토큰 | size | line-height | weight | letter-spacing | 용도 |
|------|------|-------------|--------|----------------|------|
| text-xs | 12px | 1.5 | 400 | 0 | 캡션, 라벨 |
| text-sm | 14px | 1.5 | 400 | 0 | 보조 텍스트 |
| text-base | 16px | 1.6 | 400 | 0 | 본문 |
| text-lg | 18px | 1.5 | 500 | 0 | 강조 본문 |
| text-xl | 20px | 1.3 | 600 | -0.01em | 소제목 |
| text-2xl | 24px | 1.3 | 600 | -0.015em | 섹션 제목 |
| text-3xl | 30px | 1.2 | 700 | -0.02em | 페이지 제목 |
| text-4xl | 36px | 1.1 | 700 | -0.02em | 히어로 제목 |

### 타이포 방법론

- 모듈러 스케일: Major Third (1.250) 또는 Perfect Fourth (1.333) 비율
- line-height: body 1.5~1.6, heading 1.1~1.3
- font-weight: regular(400) / medium(500) / semibold(600) / bold(700)
- letter-spacing: heading -0.02em, body 0, small-caps 0.05em

---

## 스페이싱 (8pt 그리드)

| 토큰 | 값 | 용도 |
|------|-----|------|
| space-1 | 4px | 미세 조정 (아이콘-텍스트 간격) |
| space-2 | 8px | 컴포넌트 내부 최소 패딩 |
| space-3 | 12px | 밀접한 요소 간격 |
| space-4 | 16px | 기본 패딩, 요소 간격 |
| space-6 | 24px | 섹션 내 그룹 간격 |
| space-8 | 32px | 섹션 간 간격 |
| space-12 | 48px | 주요 구분 |
| space-16 | 64px | 대형 섹션 간격 |
| space-24 | 96px | 페이지 상단/하단 여백 |
| space-32 | 128px | 히어로/랜딩 여백 |

### 스페이싱 방법론

- 기본 단위: 8pt 그리드, 4pt 하위그리드 (미세 조정용)
- 컴포넌트 내부 패딩: 최소 8pt, 터치 타겟 최소 44×44pt
- 섹션 간 여백: 최소 24pt, 주요 구분선 48pt 이상

---

## 그림자 & 엘리베이션

| 토큰 | 값 | 용도 |
|------|-----|------|
| shadow-none | none | 평면 요소 |
| shadow-sm | 0 1px 2px rgba(0,0,0,0.05) | 카드, 입력 필드 |
| shadow-md | 0 4px 6px rgba(0,0,0,0.07) | 드롭다운, 호버 카드 |
| shadow-lg | 0 10px 15px rgba(0,0,0,0.1) | 모달, 팝오버 |
| shadow-xl | 0 20px 25px rgba(0,0,0,0.15) | 토스트, 알림 |

> 다크모드: opacity 1.5배 증가

## 보더 & 라운딩

| 토큰 | 값 | 용도 |
|------|-----|------|
| radius-none | 0px | 직각 요소 |
| radius-sm | 4px | 뱃지, 태그 |
| radius-md | 8px | 버튼, 입력, 카드 |
| radius-lg | 12px | 모달, 대형 카드 |
| radius-xl | 16px | 시트, 패널 |
| radius-full | 9999px | 아바타, 원형 버튼 |

> 중첩 규칙: 내부 radius = 외부 radius - gap (음수면 0)
> border-width: 1px (기본), 2px (포커스/강조)
> border-color: neutral-200 (light), neutral-700 (dark)

---

## 모션

| 토큰 | duration | 용도 |
|------|----------|------|
| motion-instant | 0ms | 즉시 반응 |
| motion-fast | 100ms | 호버, 토글 |
| motion-normal | 200ms | 상태 전환, 드롭다운 |
| motion-slow | 300ms | 모달, 패널 |
| motion-slower | 500ms | 페이지 전환, 복잡한 애니메이션 |

> easing: ease-out (진입), ease-in (퇴장), ease-in-out (상태 전환)
> prefers-reduced-motion 시 duration 0ms로 대체
> 원칙: 의미 있는 전환만 (장식적 애니메이션 지양)

---

## 브레이크포인트 & 레이아웃

| 토큰 | 값 | 용도 |
|------|-----|------|
| screen-sm | 640px | 모바일 (세로) |
| screen-md | 768px | 태블릿 |
| screen-lg | 1024px | 소형 데스크톱 |
| screen-xl | 1280px | 데스크톱 |
| screen-2xl | 1536px | 대형 데스크톱 |

> 최대 콘텐츠 너비: 1280px (대시보드), 768px (읽기형)
> 사이드바: 240~280px 고정 또는 접이식
> 여백: 모바일 16px, 태블릿 24px, 데스크톱 32px+

---

## 컬러 시스템 방법론

- HSL 기반 정의 (테마 전환 용이)
- 시맨틱 토큰: primary, secondary, accent, success, warning, error, info
- Neutral 스케일: 50~900 (10단계)
- Surface 계층: background → surface-0 → surface-1 → surface-2 → surface-3
- 다크모드: Lightness 반전 + 채도 미세 조정, surface는 역순 매핑
- 대비비: 일반 텍스트 4.5:1 이상, 대형 텍스트 3:1 이상 (WCAG AA)

---

## 컴포넌트 토큰 매핑

<!-- /design-build가 컴포넌트 스펙 정의 시 여기에 추가 -->

| 컴포넌트 | 배경 | 텍스트 | 보더 | 라운딩 | 패딩 |
|----------|------|--------|------|--------|------|

---

## 접근성 기준 (WCAG 2.1 AA)

- 텍스트 대비: 일반 ≥4.5:1, 대형(18px+ bold / 24px+) ≥3:1
- 포커스 링: 2px solid, offset 2px, 고대비 색상
- 터치 타겟: 최소 44×44px
- 키보드: 모든 인터랙티브 요소 Tab 접근 가능
- 시맨틱 HTML 우선, aria는 보조적으로만 사용

---

## 다크모드 전략

- 방식: CSS 변수 기반 테마 전환
- Neutral: Lightness 반전 + 채도 미세 조정
- Surface: 역순 매핑 (background가 가장 어두움)
- 그림자: opacity 1.5배 증가
- 컬러: 시맨틱 토큰만 사용 시 자동 전환

---

## 참조 시스템

- Apple Human Interface Guidelines (공간, 타이포, 계층)
- Material Design 3 (컬러 시스템, 모션, 컴포넌트 상태)
- shadcn/ui + Radix UI (컴포넌트 API, 접근성, 토큰 구조)
- Linear / Vercel / Stripe (정보 밀도, 시각적 세련됨, 인터랙션)

---

## 디자인 체크리스트

- [ ] 시맨틱 토큰 사용 (하드코딩 hex/rgb 없음)
- [ ] 8pt 그리드 스페이싱 준수
- [ ] 컬러 대비비 WCAG AA (일반 4.5:1, 대형 3:1)
- [ ] 포커스 링 정의 (2px solid, offset 2px)
- [ ] 터치 타겟 44×44px 이상
- [ ] 모든 상태 정의 (default/hover/active/disabled/focus/error/loading)
- [ ] 반응형 브레이크포인트 처리
- [ ] 다크모드 토큰 매핑 (해당 시)
