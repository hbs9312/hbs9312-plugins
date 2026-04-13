---
name: create-pr
description: >
  This skill should be used when the user asks to "create pr", "make pr", "open pull request",
  "PR 올려줘", "PR 만들어줘", "풀리퀘 생성", or invokes /create-pr.
  It creates a GitHub Pull Request using the organization's PR template from the .github repository.
  The template is fetched via GitHub API (no cloning required) and filled in based on the actual changes.
  Usage: /create-pr [base-branch] [--draft] [message]
---

# Create PR Skill

Create a GitHub Pull Request using the organization's `.github` repository PR template.
The template is fetched via `gh api` and filled in based on the branch's actual changes.

## Arguments

Arguments can appear in any order. Parsing rules:

- `--draft`: Optional flag. Create the PR as a draft. Can appear anywhere.
- `--assignee {login}`: Optional. GitHub username to assign to the PR. If omitted, defaults to `@me` (the authenticated user). Use `--assignee ""` to explicitly create with no assignee.
- `[base-branch]`: Optional. The target branch to merge into. Identified as the first non-flag, non-quoted token. If omitted, defaults to the repo's default branch.
- `"[message]"`: Optional. Free-text instructions or context, enclosed in double quotes. For example, review focus areas, additional context for reviewers, or specific instructions on how to fill the template. This message is used when generating the PR title and body.

Examples:
- `/create-pr`
- `/create-pr develop`
- `/create-pr --draft`
- `/create-pr develop --draft "DB 마이그레이션 부분 집중 리뷰 부탁"`
- `/create-pr --draft "인증 로직 변경 중심으로 봐주세요" develop`
- `/create-pr --assignee octocat`
- `/create-pr develop --assignee octocat --draft`

## Procedure

### Step 1: Validate Git State

1. Confirm we are inside a git repository.
2. Get the current branch name. If on the default branch (main/master), warn the user and abort.
3. Check if there are uncommitted changes. If so, inform the user and ask whether to proceed or commit first.
4. Check if the current branch has a remote tracking branch and is pushed. If not pushed, push with `-u` flag after confirming with the user.

### Step 2: Sync with Base Branch

Before gathering PR context, ensure the current branch is up to date with the base branch to avoid merge conflicts after PR creation.

1. **Determine base branch**: Use the argument if provided, otherwise detect the repo's default branch:
   ```bash
   gh repo view --json defaultBranchRef -q '.defaultBranchRef.name'
   ```

2. **Fetch latest remote changes**:
   ```bash
   git fetch origin {base}
   ```

3. **Rebase onto the updated base branch**:
   ```bash
   git rebase origin/{base}
   ```

4. **If rebase conflicts occur**:
   - List conflicting files with `git diff --name-only --diff-filter=U`
   - Show each conflict to the user and resolve them one by one
   - After resolving each file, stage it with `git add {file}`
   - Continue the rebase with `git rebase --continue`
   - If too many conflicts or the user wants to abort, run `git rebase --abort` and inform the user

5. **Force-push the rebased branch** (since rebase rewrites history):
   ```bash
   git push --force-with-lease
   ```
   Use `--force-with-lease` instead of `--force` for safety — it will fail if someone else pushed to the branch in the meantime.

### Step 3: Gather PR Context

Collect information needed to fill in the PR:

1. **Base branch**: Already determined in Step 2.
2. **Commits**: Get all commits from the branch divergence point:
   ```bash
   git log --oneline {base}..HEAD
   ```
3. **Full diff**: Get the overall diff to understand the changes:
   ```bash
   git diff {base}...HEAD
   ```
4. **Changed files**: List all changed files:
   ```bash
   git diff --name-status {base}...HEAD
   ```

### Step 4: Fetch PR Template

#### 4-1. Check Memory

Read the auto memory file at `~/.claude/projects/{current-project}/memory/MEMORY.md` and check if a
`## PR Template` section exists with a cached template path.

#### 4-2. If Template Path Found in Memory

Use the cached information (org name, template path) to fetch the template:
```bash
gh api repos/{org}/.github/contents/{template_path} -q '.content' | base64 -d
```

If the API call fails (404, etc.), fall through to step 4-3.

#### 4-3. If Template Path NOT Found in Memory

1. **Detect the organization**:
   ```bash
   gh repo view --json owner -q '.owner.login'
   ```

2. **Search for PR template** in the org's `.github` repo. Try these paths in order:
   - `workflow-templates/PULL_REQUEST_TEMPLATE.md`
   - `workflow-templates/pull_request_template.md`
   - `.github/PULL_REQUEST_TEMPLATE.md`
   - `.github/pull_request_template.md`
   - `PULL_REQUEST_TEMPLATE.md`
   - `pull_request_template.md`

   For each path:
   ```bash
   gh api repos/{org}/.github/contents/{path} -q '.content' | base64 -d
   ```
   Use the first one that returns successfully (HTTP 200).

3. **If no template found** in the org's `.github` repo, also check the current repo itself:
   - `.github/PULL_REQUEST_TEMPLATE.md`
   - `.github/pull_request_template.md`
   - `docs/pull_request_template.md`
   - `PULL_REQUEST_TEMPLATE.md`

4. **If still no template found**, ask the user whether to:
   - Proceed without a template (generate body from commits/diff)
   - Provide a custom template path

5. **Save to memory**: Once a template is found, save the org name and template path to the project's
   `MEMORY.md` under a `## PR Template` section:
   ```markdown
   ## PR Template
   - Organization: {org}
   - Source: {org}/.github (or current repo)
   - Path: {template_path}
   ```

### Step 5: Fill In the Template

Analyze the commits and diff gathered in Step 3, then fill in the PR template intelligently:

- Replace placeholder sections (e.g., `## Summary`, `## Changes`, `## Description`) with actual content
  derived from the commits and code changes.
- If the user provided a `[message]`, incorporate it — use it as additional context for the summary, as reviewer guidance (e.g., "focus on the DB migration"), or to emphasize specific aspects of the changes.
- Keep the template's structure and section headings intact.
- If the template has checkboxes (e.g., `- [ ] Tests added`), leave them as-is for the user to check manually.
- Write in the same language as the template (if Korean, write in Korean; if English, write in English).
- Be concise but informative. Focus on **what** changed and **why**.

### Step 6: Generate PR Title

Create a concise PR title (under 70 characters) based on the changes:
- Summarize the main purpose of the PR
- Use conventional style if the repo follows it (e.g., `feat:`, `fix:`, `chore:`)
- Check recent merged PRs for title style reference:
  ```bash
  gh pr list --state merged --limit 5 --json title -q '.[].title'
  ```

### Step 7: Preview and Confirm

Present the following to the user for review using the AskUserQuestion tool:

- **Title**: The generated PR title
- **Base branch**: The target branch
- **Assignee**: The resolved assignee (e.g., `@me` or the specified username)
- **Body**: The filled-in template content (show a summary, not the full body if too long)

Options:
- "Create PR (Recommended)" — proceed with the generated content
- "Edit title" — let the user provide a custom title
- "Edit body" — let the user modify the body before creating
- "Cancel" — abort

### Step 8: Create the Pull Request

```bash
gh pr create --base {base_branch} --title "{title}" --body "$(cat <<'EOF'
{filled_template_body}
EOF
)" --assignee {assignee} [--draft]
```

- Add `--draft` flag if the user passed `--draft` argument.
- `{assignee}` is `@me` by default, or the value from `--assignee` if explicitly provided. If the user passed `--assignee ""`, omit the `--assignee` flag entirely.

After successful creation, display the PR URL to the user.

## Guidelines

- Always use `gh api` to fetch templates — never clone the `.github` repo.
- The `base64 -d` command decodes the GitHub API's base64-encoded file content.
- If the GitHub API rate limit is hit, inform the user and suggest trying again later.
- Respect the template's original formatting and structure when filling it in.
- Do not modify checkbox items — leave them for the user to manage.
- If the template contains sections that don't apply to the current changes, write "N/A" or leave them empty rather than removing them.
- The PR body content should be factual and based on the actual diff — do not fabricate changes.
