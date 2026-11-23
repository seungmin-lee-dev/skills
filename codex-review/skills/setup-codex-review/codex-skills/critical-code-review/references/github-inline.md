# GitHub inline review notes

Read this for GitHub PR review, ready-to-post payloads, or authorized posting. This is an operator procedure using `gh api`, not an automatic posting script. All examples are synthetic; replace identifiers with the verified target. Never post to an example or real PR just to test this procedure.

## 1. Pin identity, scope, and evidence (read-only)

Use explicit `OWNER/REPO` and `PR_NUMBER` in every request; do not rely on the current checkout or CLI defaults.

```sh
gh api repos/OWNER/REPO/pulls/PR_NUMBER
gh api --paginate repos/OWNER/REPO/pulls/PR_NUMBER/files
gh api -H "Accept: application/vnd.github.v3.diff" repos/OWNER/REPO/pulls/PR_NUMBER
gh api --paginate repos/OWNER/REPO/pulls/PR_NUMBER/comments
gh api --paginate repos/OWNER/REPO/pulls/PR_NUMBER/reviews
```

Confirm `html_url`, `number`, `base.repo.full_name`, `base.sha`, `head.sha`, and `head.repo` (forks are possible). Record the full reviewed head SHA and base. Read the PR body and material linked requirements using their exact repository/issue IDs; disclose inaccessible sources. Fetch code at that SHA when needed; the local worktree is not automatically the PR head. Read only the bounded evidence needed by the main skill.

Get the PR head again after diff/evidence acquisition. If it changed during reads, do not attribute the mixed snapshot to either head: reacquire/review only within authorization and capability. Missing/truncated file patches require the complete diff and pinned source; binary files or unverifiable coordinates yield summary-only findings. Paginate reads, and disclose API limits instead of assuming all files were returned. Compare the existing review bodies and inline threads semantically to avoid duplicating a root cause already noted.

## 2. Derive an actual changed-line anchor

Parse each file and each `@@ -oldStart,oldCount +newStart,newCount @@` hunk with separate old/new counters (omitted count means 1):

| Hunk row | Coordinate | Advance |
|---|---|---|
| context (` `) | Neither is a changed-line anchor for this skill | old +1, new +1 |
| deletion (`-`) | old counter, `LEFT` | old +1 only |
| addition (`+`) | new counter, `RIGHT` | new +1 only |
| `\ No newline at end of file` | None | Neither |

File headers `---`/`+++` are not hunk rows. Reset counters at every hunk; never use patch row numbers as file lines. Verify the API `path` against the PR files entry, including renamed files (`filename` is the current path; `previous_filename` helps inspect the old source). Do not derive paths by naively splitting quoted diff headers. For deletion-only causes, use the old line at `LEFT` with the verified API path only when the chosen surface supports it; otherwise summary-only. A removed file still needs a path verified from the PR file metadata. Keep one smallest causal changed line per finding; avoid multiline ranges unless necessary and fully verified.

For example, in synthetic `src/ratio.py`:

```diff
@@ -1,2 +1,2 @@
 def ratio(total, count):
-    return total / count if count else 0
+    return total / count
```

The added return is `line: 2, side: RIGHT`; the deleted return is `line: 2, side: LEFT`. For input `(8, 0)`, the addition demonstrably raises instead of returning zero. Choose one causal note, not both. If the PR only deletes an authorization guard, use its verified old line/LEFT, never an invented surviving line.

## 3. Prepare, gate, then publish once

Keep notes local unless explicit authorization covers comments on this exact PR. For an Orca-woken bot, use its supplied `orca bot reply --note ...` route and anchor syntax instead; do not run the GitHub write commands below as a second route. Unsupported anchors stay in that reply's summary.

Default to one submitted comment review for the confirmed, material findings. Write a UTF-8 JSON file with a structured file writer (no interpolated shell body). For the synthetic hunk above, the shape is:

```json
{
  "commit_id": "1111111111111111111111111111111111111111",
  "event": "COMMENT",
  "body": "검토한 head와 범위, 요구사항 대조 및 검증 한계 요약",
  "comments": [
    {
      "path": "src/ratio.py",
      "line": 2,
      "side": "RIGHT",
      "body": "[High][Correctness][merge blocker] 0개 입력에서 예외 발생\nconfidence: 9/10; introduced. count=0이면 기존 0 반환 대신 ZeroDivisionError로 요청이 실패합니다. 최소 수정: 0 처리 가드를 복구하세요."
    }
  ]
}
```

`commit_id` belongs at the review level; `path`, `line`, `side`, `body` belong to each comment. Use modern `line`/`side`, not deprecated diff `position`. Do not omit `event` (that creates a pending, unpublished review). Do not submit an empty review merely to show activity on a clean PR.

Immediately before the write, GET the exact PR again and compare `head.sha` to the full reviewed SHA; verify the payload's `commit_id` matches it. A mismatch or failed read stops publication and makes notes stale. Re-review only if authorized for the new head and able to obtain fresh evidence. **Only after that comparison succeeds and all skill gates pass**, execute the prepared write once:

```sh
gh api --method POST -H "Accept: application/vnd.github+json" repos/OWNER/REPO/pulls/PR_NUMBER/reviews --input review.json
```

If the surface requires one individual inline note instead, the alternative is `POST repos/OWNER/REPO/pulls/PR_NUMBER/comments` with top-level `body`, `commit_id`, `path`, `line`, `side`. This creates an inline review comment, not an issue/timeline comment; it has no `event` field. Use **either** this or the batch review for a finding, never both. Both REST endpoints support LEFT/RIGHT; a wrapper that exposes only RIGHT does not. Do not send a review-level payload to the individual endpoint. Neither route authorizes `APPROVE` or `REQUEST_CHANGES`.

## 4. Verify delivery and stop safely

Save the response IDs/URLs. For a batch review, GET `repos/OWNER/REPO/pulls/PR_NUMBER/reviews/REVIEW_ID` and paginate `repos/OWNER/REPO/pulls/PR_NUMBER/reviews/REVIEW_ID/comments`; verify `state: COMMENTED`, `submitted_at`, reviewed `commit_id`, and each expected note's body/path/side/line. For an individual note, GET `repos/OWNER/REPO/pulls/comments/COMMENT_ID` and verify its body/coordinates/commit. Confirm notes are visible via returned links and not merely pending drafts. Re-read PR head after writing and report if it moved; the API has no atomic compare-and-post guard, so stop further notes and disclose that race rather than claiming the newest head was reviewed.

On a timeout/connection loss or ambiguous response, do not POST again. Inspect exact-PR reviews and comments (including pending reviews visible to this actor), matching head, body, root cause, and coordinates, before deciding whether the write succeeded. If visibility or delivery is still uncertain, stop and report uncertainty with saved IDs. A definite validation/auth/rate-limit error is not a reason to change coordinates, permissions, or event blindly: diagnose it, recheck head/deduplication and authorization before any justified retry. Never publish test comments to exercise the API.

Endpoint field references: [create a review](https://docs.github.com/en/rest/pulls/reviews#create-a-review-for-a-pull-request), [create a review comment](https://docs.github.com/en/rest/pulls/comments#create-a-review-comment-for-a-pull-request).
