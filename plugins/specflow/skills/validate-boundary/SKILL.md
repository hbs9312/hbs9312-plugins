---
name: validate-boundary
description: 문서가 자신의 영역 경계를 침범하지 않는지 검증합니다. "경계 검증" 요청 시 사용.
argument-hint: [검증할 문서 파일 경로]
disable-model-invocation: true
context: fork
allowed-tools: Read Grep Glob Write
model: claude-opus-4-6
effort: high
---

# 경계 침범 검증 (V3)

ultrathink

문서 간 책임 경계의 감시자입니다. `context: fork` 격리 실행.

## 컨텍스트

- `${CLAUDE_SKILL_DIR}/../../context/conventions.md` 를 Read로 읽으세요
- `${CLAUDE_SKILL_DIR}/../../context/glossary.md` 를 Read로 읽으세요 — 도메인 용어 예외 판단에 필요

## 입력: $ARGUMENTS 문서를 Read. ID 접두사로 유형 판별.

## 경계 정의

### FS에서 기술 침범 탐지
키워드: PostgreSQL, MySQL, MongoDB, Redis, Elasticsearch, Pub/Sub, Kafka, RabbitMQ, WebSocket, gRPC, GraphQL, Cloud Run, GCS, S3, EC2, Lambda, Kubernetes, Docker, Terraform, nginx, 테이블, 스키마, 인덱스, FK, 마이그레이션, 커넥션풀, 엔드포인트, REST, HTTP, POST, GET, DELETE, PUT, 벡터, 임베딩, 큐, 토픽, 워커, 배치, 크론잡, SHA256, JWT, OAuth
★ glossary.md 도메인 용어는 예외

### TS에서 비즈니스 침범 탐지
패턴: "사용자가 원하는", "비즈니스 목표는", "이 기능이 필요한 이유", UX 카피("~해 주세요"), 사용자 감정("혼란을 느끼지 않도록", "직관적으로")

### WF에서 디자인 침범 탐지
패턴: #hex, rgb(, hsl(, $color-*, font-size, font-weight, Npx, $text-*, Nms, ease-in, ease-out, transition, animation, $spacing-*

## 출력

```yaml
검증 유형: V3 (경계 침범)
findings:
  - id: "V3-001"
    severity: warning
    location: "{섹션}"
    content: "{침범 원문}"
    violation: "{경계 유형}"
    glossary_exception: false
    suggestion: "{수정 제안}"
summary:
  glossary_exceptions_applied: {N}
```

## 저장: specs/reviews/{문서ID}-V3-{timestamp}.md
