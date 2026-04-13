# ghflow

GitHub 이슈·PR·리뷰를 잇는 워크플로우 스킬 모음.

## 워크플로우

```
pick-issue → draft-pr → (작업) → create-pr → review-pr → clear-issue
                ↑
          create-issue (필요 시)
```

## 스킬

| 스킬 | 트리거 | 역할 |
|---|---|---|
| `create-issue` | `/create-issue`, "이슈 만들어줘" | 조직 이슈 템플릿에 맞춰 GitHub 이슈 생성 |
| `pick-issue` | `/pick-issue`, "내 이슈 보여줘" | assigned 이슈 목록 → 선택 → 프로젝트 메모리에 저장 |
| `draft-pr` | `/draft-pr`, "Draft PR 만들어줘" | 이슈 번호 기반으로 브랜치+빈 커밋+Draft PR 생성, 이슈 연결 |
| `create-pr` | `/create-pr`, "PR 만들어줘" | 조직 PR 템플릿(`.github` 레포)을 가져와 PR 생성 |
| `review-pr` | `/review-pr`, "PR 리뷰 확인" | PR 리뷰 댓글 가져와 코드와 함께 검토 |
| `clear-issue` | `/clear-issue`, "이슈 정리" | pick-issue가 저장한 현재 작업 이슈 메모리 삭제 |

## 의존성

- `gh` CLI 인증 필요 (`gh auth login`)
- 조직 템플릿 사용 시 `.github` 레포 접근 권한 필요
