---
name: create-issue
description: >
  GitHub 이슈를 조직 템플릿에 맞춰 생성하는 스킬. 사용자가 '이슈 만들어줘', '이슈 생성', '버그 리포트',
  '기능 제안', '작업 등록', 'create issue', 'github issue', '/create-issue' 등을 말하면 트리거한다.
  이슈와 관련된 요청이면 적극적으로 이 스킬을 사용할 것.
  Usage: /create-issue [-y] [--refresh] <사용자 의도>
---

# Create Issue

GitHub 이슈를 조직의 이슈 템플릿에 맞춰 생성한다. 사용자의 의도를 파악하여 적절한 템플릿을 선택하고, 내용을 채워 이슈를 발행한다.

## 사용법

```
/create-issue <사용자 의도>
/create-issue -y <사용자 의도>      # 확인 없이 즉시 생성
```

## 실행 흐름

### 1. 템플릿 로딩

이슈 템플릿은 프로젝트별로 캐싱된다. 아래 순서를 따른다.

**캐시 경로 결정:**
```bash
# 현재 레포의 org/repo 식별
REPO_ID=$(gh repo view --json nameWithOwner --jq '.nameWithOwner')
# 예: "soy-media/velvetalk-fe"
```

캐시 파일 경로: `~/.claude/cache/issue-templates/<org>/<repo>/templates.json`

**캐시 확인:**
- 캐시 파일이 존재하면 → 그대로 사용
- 캐시 파일이 없으면 → 원격에서 가져와 캐싱

**원격 템플릿 가져오기:**
```bash
# 1단계: 현재 레포에 템플릿이 있는지 확인
gh api repos/<org>/<repo>/contents/.github/ISSUE_TEMPLATE 2>/dev/null

# 2단계: 없으면 조직의 .github 레포에서 가져오기
gh api repos/<org>/.github/contents/.github/ISSUE_TEMPLATE 2>/dev/null
```

각 템플릿 파일의 content를 base64 디코딩하여 `templates.json`으로 저장한다.

**templates.json 형식:**
```json
{
  "source": "soy-media/.github",
  "fetched_at": "2026-03-12T...",
  "templates": [
    {
      "filename": "bug_report.md",
      "name": "버그 리포트",
      "title_prefix": "[BUG] ",
      "labels": ["bug"],
      "body": "## 🐛 버그 설명\n..."
    },
    {
      "filename": "feature_request.md",
      "name": "기능 제안",
      "title_prefix": "[FEAT] ",
      "labels": ["enhancement"],
      "body": "## 🚀 기능 제안 배경\n..."
    },
    {
      "filename": "task.md",
      "name": "작업 (Task)",
      "title_prefix": "[TASK] ",
      "labels": ["task"],
      "body": "## 📌 작업 개요\n..."
    }
  ]
}
```

캐시를 강제로 갱신하고 싶으면 `--refresh` 인자를 사용한다.

### 2. 사용자 의도 파악 및 템플릿 선택

사용자의 입력에서 다음을 판단한다:

| 판단 항목 | 설명 |
|-----------|------|
| 템플릿 종류 | bug_report / feature_request / task 중 하나. 의도에서 자연스럽게 추론한다 |
| 제목 | 템플릿의 title_prefix 뒤에 붙을 간결한 제목 |
| 본문 내용 | 템플릿의 각 섹션을 채울 내용 |
| 라벨 | 템플릿 기본 라벨 + 사용자가 추가로 지정한 라벨 |
| Assignees | 기본: 현재 사용자 본인. 별도 지시가 있으면 추가 |

**컨텍스트가 부족한 경우:**

사용자의 입력만으로 템플릿 섹션을 채우기 어렵다면, 부족한 부분을 구체적으로 짚어서 되물어야 한다. 예시:

- 템플릿 종류를 특정할 수 없는 경우: "버그 리포트, 기능 제안, 작업 중 어떤 유형인가요?"
- 버그인데 재현 절차가 없는 경우: "재현 절차를 알려주시겠어요?"
- 기능 제안인데 배경이 불명확한 경우: "이 기능이 필요한 배경이나 해결하려는 문제가 뭔가요?"

단, 모든 섹션을 강제로 채울 필요는 없다. 사용자가 제공한 정보로 핵심 섹션을 채울 수 있으면 나머지는 비워두거나 적절히 생략한다.

### 3. 이슈 미리보기 및 확인

`-y` 플래그가 **없으면**, 이슈를 생성하기 전에 다음 형식으로 미리보기를 보여주고 확인을 받는다:

```markdown
## 이슈 미리보기

- **레포**: soy-media/velvetalk-fe
- **유형**: 기능 제안
- **제목**: [FEAT] 채팅방 메시지 검색 기능
- **라벨**: enhancement
- **Assignees**: hbs9312

---

### 본문

## 🚀 기능 제안 배경

채팅방에서 이전 대화 내용을 찾기 어려움...

## 💡 제안하는 기능

...

---

> 이대로 이슈를 생성할까요?
```

사용자가 수정을 요청하면 반영 후 다시 미리보기를 보여준다.

`-y` 플래그가 **있으면**, 미리보기 없이 바로 생성한다.

### 4. 이슈 생성

```bash
gh issue create \
  --repo <org>/<repo> \
  --title "<title_prefix><제목>" \
  --body "<본문>" \
  --label "<라벨1>,<라벨2>" \
  --assignee "<사용자>,<추가 assignee>"
```

생성 후 이슈 URL을 사용자에게 알려준다.

### 5. Assignees 규칙

- 기본적으로 현재 GitHub 사용자를 assignee로 추가한다: `gh api user --jq '.login'`
- 사용자가 "OOO에게 할당해줘", "@username 추가" 등으로 지시하면 해당 사용자를 추가한다
- 본인을 제외하라는 명시적 지시가 없는 한, 본인은 항상 포함한다

## 주의사항

- 이슈 생성 대상 레포는 기본적으로 현재 작업 디렉토리의 레포이다. 사용자가 다른 레포를 지정하면 해당 레포에 생성한다.
- 템플릿 캐시를 갱신하려면 `--refresh` 인자를 전달하거나, 캐시 파일을 직접 삭제한다.
- 템플릿의 YAML frontmatter(name, about, title, labels, assignees)는 파싱하여 메타데이터로 활용하고, 본문은 frontmatter 이후의 마크다운 부분이다.
