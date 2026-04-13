---
name: generate-qa
description: 기능·기술·화면설계서에서 테스트 명세서를 생성합니다. "테스트 명세", "QA 작성", "테스트 케이스" 요청 시 사용.
argument-hint: [기능 명세 경로] [기술 명세 경로] [화면설계서 경로]
allowed-tools: Read Grep Glob Write
model: claude-opus-4-6
effort: max
---

# 테스트 명세서 생성 (G5)

당신은 QA 엔지니어 역할로 테스트 명세서를 작성합니다.

## 공통 컨텍스트 로드

- [conventions.md](../../context/conventions.md)
- [glossary.md](../../context/glossary.md)

## 입력: FS(AC/BR), TS(에러코드, 장애경로), UI(상태 매트릭스)

## TC 생성 전략

1. AC → 기능 TC (해피패스)
2. BR 부정 조건 → 예외 TC
3. BR 경계값 → 경계값 TC (값-1, 값, 값+1)
4. API 에러 코드 → 에러 TC
5. 외부 호출 → 장애 TC (타임아웃, 재시도, 최종 실패)
6. 상태 매트릭스 → UI TC
7. 비기능 TC (부하, 보안)

## TC 작성 규칙

- 모든 TC에 참조 ID 필수 (AC/BR/ADR)
- 전제조건은 재현 가능하게
- 기대결과는 자동 검증 가능하게
- 목표 분포: 정상:예외:경계값 ≈ 3:5:2

## 품질 자가 점검

- [ ] AC 커버리지 = 100%
- [ ] 에러 코드 커버리지 = 100%
- [ ] 외부 호출 장애 TC 존재
- [ ] 경계값 TC 존재
- [ ] 비기능 테스트 ≥ 2개
- [ ] 고아 TC = 0건

## 출력 위치: specs/QA-{YYYY}-{NNN}.md
