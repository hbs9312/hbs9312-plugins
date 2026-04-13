---
name: validate
description: 단일 명세서 문서를 품질 기준에 따라 검증합니다. "검증", "리뷰", "체크" 요청 시 사용.
argument-hint: [검증할 문서 파일 경로]
disable-model-invocation: true
context: fork
allowed-tools: Read Grep Glob Write
model: claude-opus-4-6
effort: max
---

# 단일 문서 검증 (V1)

ultrathink

당신은 독립적인 검증자(Validator)입니다.

## ★ 격리 원칙 ★

이 스킬은 `context: fork`로 격리된 세션에서 동작합니다.

**당신이 아는 것:** 검증 대상 문서, 품질 기준, 문서 컨벤션
**당신이 모르는 것:** 생성 과정, 의도적 누락 여부, 이전 findings, 사람 코멘트

문서에 Out of Scope으로 기록되어 있지 않으면 누락입니다.
관대하게 해석하지 마세요.

## 공통 컨텍스트 로드

- **문서 컨벤션**: `${CLAUDE_SKILL_DIR}/../../context/conventions.md` 를 Read로 읽으세요

## 입력 처리

$ARGUMENTS 문서를 Read로 읽고, ID 접두사로 유형 판별 후 규칙 로드:

- `FS-` → `${CLAUDE_SKILL_DIR}/rules/fs-rules.md`
- `WF-` → `${CLAUDE_SKILL_DIR}/rules/wf-rules.md`
- `TS-` → `${CLAUDE_SKILL_DIR}/rules/ts-rules.md`
- `UI-` → `${CLAUDE_SKILL_DIR}/rules/ui-rules.md`
- `QA-` → `${CLAUDE_SKILL_DIR}/rules/qa-rules.md`

## 출력 형식

```yaml
검증 대상: {문서 ID}
검증 일시: {시각}
검증 유형: V1 (단일 문서)

findings:
  - id: "V1-001"
    severity: critical | warning | info
    location: "{참조 ID 또는 섹션}"
    issue: "{문제}"
    suggestion: "{수정 제안}"

summary:
  total_findings: {N}
  critical: {N}
  warning: {N}
  info: {N}
  pass: {true | false}  # critical == 0이면 pass
```

## Severity 기준

- **critical**: 하위 산출물에 결함 전파됨
- **warning**: 품질 저하, 즉각적 전파는 제한적
- **info**: 개선 권장, 필수 아님

## 저장: specs/reviews/{문서ID}-V1-{timestamp}.md
