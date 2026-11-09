---
name: pr-review
description: Review a GitHub pull request at a chosen depth and post the result to GitHub — a short summary of what the PR does plus inline comments on the exact lines of each finding, every comment prefixed so it is recognizable as written by Claude.
argument-hint: <pr-number> [low|medium|high|xhigh|max] [project-dir]
disable-model-invocation: true
---

# PR review → GitHub comments

Arguments: `$ARGUMENTS`

- **1st** (required): PR number, e.g. `42` (a `#42` or a full PR URL also works).
- **2nd** (optional, default `medium`): review level `low | medium | high | xhigh | max`.
- **3rd** (optional): project directory (`mvu-platform`, `mvu-platform-app`, `mvu-website`) when the
  current directory is not inside the repo the PR belongs to. If the cwd is the workspace root and no
  project is given, ask which project.

Invoking this skill **is** the authorization to post to the PR — don't ask again before posting.
Never approve, request changes, merge, push or edit the PR; only post one `COMMENT` review.

## Comment prefix

Every text posted to GitHub (the summary and each inline comment) starts with:

```
🤖 **Claude:**
```

## 1. Gather the PR

Run all `gh`/`git` commands from the project directory.

```bash
gh pr view <N> --json number,title,body,url,author,baseRefName,headRefName,headRefOid,files,additions,deletions
gh pr diff <N>
git fetch origin pull/<N>/head        # read full files at the head commit without touching the working tree
git show <headRefOid>:<path>          # surrounding context when the diff alone isn't enough
```

Also read the project's `CLAUDE.md` — its conventions (e.g. ArchUnit rules in `mvu-platform`) are
review criteria.

## 2. Review at the requested level

| Level    | What to report                                                                                           | Typical count |
|----------|----------------------------------------------------------------------------------------------------------|---------------|
| `low`    | Only high-confidence, real defects: bugs, crashes, data loss, security holes.                            | 0–3           |
| `medium` | Above, plus likely bugs, missing edge cases/error handling, broken project conventions.                  | 0–6           |
| `high`   | Above, plus maintainability, simplification, performance, test gaps; lower-confidence issues allowed.    | 0–12          |
| `xhigh`  | Like `high`, but split the changed files across parallel subagents (one per area), then verify each finding yourself. | any |
| `max`    | Like `xhigh`, plus an independent adversarial verification pass per finding; drop anything not confirmed. | any           |

Always:

- Verify each finding against the actual code at the head commit before keeping it — no guesses
  dressed up as facts. Mark lower-confidence findings as such ("Possibly …").
- **Cross-project impact:** if the PR changes a platform REST endpoint (path, params, DTO shape), grep
  the sibling projects (`../mvu-website`, `../mvu-platform-app`) for that path and report breakage.
- Zero findings is a valid result — don't invent nitpicks to fill the table.

## 3. Map findings to diff lines

Inline comments can only target lines that appear in the PR diff. For each finding pick `path` and
`line` = line number in the **new** file (`side: "RIGHT"`) of an added or context line in a hunk.
For a multi-line range add `start_line` (+ `start_side: "RIGHT"`), both inside the same hunk.
A finding whose line is not in the diff goes into the summary instead (one bullet with `path:line`).

## 4. Post one review

Write the payload to a JSON file in the scratchpad (not the repo), then:

```bash
gh api repos/{owner}/{repo}/pulls/<N>/reviews --method POST --input <payload.json>
```

(`gh` fills in `{owner}/{repo}` from the current repo.)

Payload:

```json
{
  "commit_id": "<headRefOid>",
  "event": "COMMENT",
  "body": "🤖 **Claude:** <summary>",
  "comments": [
    { "path": "src/main/java/…/FooService.java", "line": 87, "side": "RIGHT",
      "body": "🤖 **Claude:** **Bug** — <what is wrong, why, concrete fix>" }
  ]
}
```

**Summary (`body`) — keep it short:** 2–4 sentences or bullets on *what the PR does* (not a
file-by-file list), then one line with the level and finding count, e.g.
`Review level: medium · 3 findings (1 bug, 2 suggestions)`. Add the "not on a diff line" bullets only
if there are any.

**Inline comment:** start with a severity tag (`**Bug**`, `**Security**`, `**Risk**`, `**Suggestion**`,
`**Nit**`), then 1–3 sentences: the problem, why it matters, the fix. Use a GitHub ```` ```suggestion ````
block when the fix is a small, exact replacement of the commented lines.

If the API rejects a comment (HTTP 422, usually a line not in the diff), move that finding into the
summary and post again — don't drop it silently.

## 5. Report back

Tell the user in the terminal: the review URL (`html_url` from the response), and a one-line list of
the findings posted. Nothing else.
