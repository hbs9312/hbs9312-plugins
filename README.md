# hbs9312-plugins

hbs9312가 관리하는 Claude Code 플러그인 통합 마켓플레이스.

## 설치

```bash
/plugin marketplace add hbs9312/hbs9312-plugins
/plugin install specflow@hbs9312-plugins
/plugin install frontflow@hbs9312-plugins
/plugin install backflow@hbs9312-plugins
/plugin install meeting-prep@hbs9312-plugins
/plugin install ghflow@hbs9312-plugins
```

## 포함된 플러그인

| 플러그인 | 버전 | 설명 |
|---|---|---|
| [specflow](./plugins/specflow) | 1.3.0 | 에이전트 기반 개발 명세서 생성·검증·수정 워크플로우. PRD → 기능/기술 명세, 와이어프레임, 화면설계서, 테스트 명세, 구현 계획 자동 생성 |
| [frontflow](./plugins/frontflow) | 1.2.0 | 프론트엔드 구현 워크플로우. Figma → 코드를 바텀업으로 구축 |
| [backflow](./plugins/backflow) | 0.1.0 | specflow 산출물 기반 백엔드 구현 워크플로우. 스키마 → 리포지토리 → 서비스 → 컨트롤러 → 미들웨어 → 통합 |
| [meeting-prep](./plugins/meeting-prep) | 0.1.0 | 기획서 분석과 구현 현황을 병렬 분석하여 회의 준비 문서를 자동 생성 |
| [ghflow](./plugins/ghflow) | 0.1.0 | GitHub 이슈·PR·리뷰 워크플로우 스킬 모음 (create-issue / pick-issue / draft-pr / create-pr / review-pr / clear-issue) |

## 구조

```
hbs9312-plugins/
├── .claude-plugin/
│   └── marketplace.json
└── plugins/
    ├── specflow/
    ├── frontflow/
    ├── backflow/
    ├── meeting-prep/
    └── ghflow/
```

각 플러그인의 상세 문서는 해당 디렉토리의 README를 참고하세요.

## 마이그레이션 안내

이 레포는 기존 4개 독립 레포(`hbs9312/specflow`, `hbs9312/frontflow`, `hbs9312/backflow`, `hbs9312/meeting-prep`)를 하나로 통합한 것입니다. 기존 마켓플레이스를 사용 중이라면:

```bash
/plugin marketplace remove specflow
/plugin marketplace remove frontflow
/plugin marketplace remove backflow
/plugin marketplace remove meeting-prep
/plugin marketplace add hbs9312/hbs9312-plugins
```

## License

각 플러그인의 라이선스를 따릅니다.
