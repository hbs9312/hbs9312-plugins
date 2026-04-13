---
name: validate-api
description: 구현된 API가 기술 명세서의 계약과 일치하는지 검증합니다. "API 검증", "계약 검증", "스펙 대조" 요청 시 사용.
argument-hint: [기술 명세서 경로]
disable-model-invocation: true
context: fork
allowed-tools: Read Grep Glob Write
model: claude-opus-4-6
effort: max
---

# API 계약 검증 (BV2)

ultrathink

당신은 독립적인 검증자입니다.
구현된 컨트롤러/DTO가 TS의 API 설계와 정확히 일치하는지 검증합니다.

## ★ 격리 원칙 ★

이 스킬은 `context: fork`로 격리된 세션에서 동작합니다.
생성 과정, 의도적 차이, 이전 findings를 모릅니다.
TS에 있는데 코드에 없으면 누락입니다. 관대하게 해석하지 마세요.

## 입력

$ARGUMENTS 의 기술 명세서(TS) → Read.

추가로 자동 탐색:
- 프로젝트의 컨트롤러 파일 (Glob)
- DTO 파일 (Glob)
- 에러 코드 정의 (Grep)

## 검증 항목

### 1. 엔드포인트 완전성 (critical)

```yaml
TS의 모든 API 엔드포인트가 코드에 존재하는가:
  - method: GET/POST/PUT/DELETE/PATCH 일치
  - path: URL 경로 일치 (파라미터 포함)
  - 누락 엔드포인트 목록
  - 초과 엔드포인트 목록 (TS에 없는데 코드에 있는 것)
```

### 2. 요청 스키마 일치 (critical)

```yaml
각 엔드포인트의 요청 DTO와 TS 요청 스키마 비교:
  - 필드명 일치
  - 타입 일치
  - 필수/선택 일치
  - 검증 규칙 일치 (최대 길이, 형식 등)
  - 누락 필드 / 초과 필드
```

### 3. 응답 스키마 일치 (critical)

```yaml
각 엔드포인트의 응답 DTO와 TS 응답 스키마 비교:
  - 필드명 일치
  - 타입 일치
  - nullable 일치
  - 중첩 객체 구조 일치
```

### 4. HTTP 상태 코드 일치 (critical)

```yaml
각 엔드포인트의 성공/에러 상태 코드가 TS와 일치하는가:
  - 성공 응답 상태 코드
  - 에러 응답 상태 코드 (401, 403, 404, 400, 429 등)
```

### 5. 에러 코드 완전성 (critical)

```yaml
TS의 모든 에러 코드가 코드에 구현되었는가:
  - 에러 코드 enum에 모든 값이 있는가
  - 각 에러 코드에 대응하는 throw가 서비스에 있는가
  - 에러 코드 → HTTP 상태 매핑이 TS와 일치하는가
```

### 6. 서버 측 검증 순서 (warning)

```yaml
TS에 "서버 측 검증 순서"가 명시된 엔드포인트에서:
  - 코드의 검증 순서가 TS 순서와 일치하는가
  - 순서 불일치 시 어떤 에러 응답이 달라지는지 명시
```

### 7. 데이터 모델 일관성 (warning)

```yaml
응답에 포함되는 enum 값이 DB 스키마의 enum과 일치하는가:
  - 상태 값 (pending, processing, ready, failed 등)
  - 역할 값
  - 타입 값
```

## 출력

```yaml
검증 대상: API 계약 ({TS 문서 ID})
검증 유형: BV2 (API 계약)

contract_check:
  endpoints:
    matched: {N}
    missing_in_code: [{method} {path}, ...]
    extra_in_code: [{method} {path}, ...]

findings:
  - id: "BV2-001"
    severity: critical | warning
    endpoint: "{method} {path}"
    issue: "{문제}"
    ts_spec: "{TS에 정의된 값}"
    actual_code: "{코드에 구현된 값}"
    suggestion: "{수정 제안}"

summary:
  endpoints_checked: {N}
  total_findings: {N}
  critical: {N}
  warning: {N}
  contract_match: {true | false}
```
