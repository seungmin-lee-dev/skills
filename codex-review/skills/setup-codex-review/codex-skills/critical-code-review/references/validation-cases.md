# Standalone review dry-run cases

Maintainer fixtures for instruction changes. Apply SKILL.md and github-inline.md to each request; record the actual report/payload decision and compare with the expected outcome. All repositories, PRs, SHAs, and code below are synthetic. Do not send any write request. These cases are a manual behavioral walkthrough, not proof of live GitHub delivery or installed skill discovery.

Shared target unless overridden: `example/review-fixture` PR 123, reviewed/current head `1111111111111111111111111111111111111111`, no existing notes, complete patch. File metadata supplies the displayed path. Review authorization and API capabilities are stated per case.

## A. Actual changed RIGHT line

Request: “Review PR 123 and comment on confirmed material defects.” Requirement: empty sample counts must return zero, used by the request handler. File `src/ratio.py`:

```diff
@@ -1,2 +1,2 @@
 def ratio(total, count):
-    return total / count if count else 0
+    return total / count
```

Trace `(8, 0)` through both versions. Expect one introduced finding tagged Correctness and Spec (one root cause), High/merge blocker, confidence 9–10, restoring the zero guard as smallest fix. Prepared COMMENT review anchors `src/ratio.py:2 RIGHT` and uses the reviewed commit_id. The deleted row is not a second note. In a real authorized run, publish only after the head check and verify delivery; this fixture remains dry-run.

## B. Deletion-only LEFT, supported and unsupported

Request authorizes notes. Contract: only the item's owner may read its secret. Route has already authenticated the user; no other ownership gate exists. File `src/secret.py`:

```diff
@@ -1,4 +1,2 @@
 def read_secret(user, item):
-    if user.id != item.owner_id:
-        raise PermissionError()
     return item.secret
```

Trace a signed-in non-owner. Expect one introduced Security finding, Critical/merge blocker, confidence 9–10, restoring the ownership check. On the REST review endpoints, anchor old line 2 LEFT (or line 3 if that is the precisely explained cause), never new line 2 RIGHT: it is context. With a RIGHT-only wrapper, keep this summary-only; do not invent an anchor. A suppression accepting public metadata cannot suppress this secret exposure, even if phrased broadly.

## C. Stale head before publication

Use A, but the immediate pre-write head read returns `2222222222222222222222222222222222222222`. Request authorized only the original head. Expect zero writes, notes marked stale, no rebasing coordinates and no claim the new head was reviewed. A failed pre-write GET also stops posting. If the head changes only after a successful write, report the race and returned IDs, stop further notes, and do not retry.

## D. Local diff or plain review request

Use A first as an uncommitted local diff without a PR, then as “Review PR 123” without posting authorization. Expect the actionable report and ready-to-post note, zero writes in both cases. The local case says PR/head is unavailable as applicable and records base/worktree state. GitHub credentials or an existing branch name do not grant permission; no APPROVE/REQUEST_CHANGES.

## E. Evidence gap

Request authorizes notes, but linked acceptance issue is inaccessible and only a call `cache.get(key)` is provided; cache implementation, expiry behavior, and failure trace are unavailable. Expect the requirement/structural evidence gaps, no asserted Spec failure, no confirmed stale-cache finding, and zero inline notes. A plausible stale-cache risk stays in the report with confidence at most 6 and the exact missing evidence. Separate tests read from tests executed; do not infer a passing run.

## F. Small clean PR

Requirement: document the existing zero-count behavior. Diff changes only `README.md` from “Return the ratio.” to “Return zero when count is zero.” Source still has A's original zero guard; the relevant test is read but not run. Expect no actionable findings, explicit requirement match, read/not-run distinction, bounded residual risk, and a short local merge recommendation. No panel debate, nits, empty comment review, or write even if notes were authorized.

## G. Missing requirement without an anchor

Requirement: expose a `/ready` health route as well as the implemented `/live` route. Complete reviewed route registry only adds `/live`; tests cover `/live` only. Expect one confirmed introduced Spec omission, with severity/disposition justified by the documented rollout contract. Cite the inspected registry location in the report, but keep it summary-only when no changed line causes the absence of `/ready`; do not attach the unrelated `/live` addition just to post.

## H. Duplicate, ambiguous write, and Orca route

For A, an existing note already describes count=0 at the same root cause: skip a duplicate. If a prior POST timed out, inspect reviews/inline comments at the reviewed head before any retry; an inconclusive read means stop and report uncertainty. If an Orca PR event supplied `orca bot reply --note ...`, prepare only that route's supported note (or summary for unsupported anchors), never a second GitHub POST.

## I. Suppression and pre-existing attribution

Ignore file documents unauthenticated access to a public `/version` metadata endpoint: a matching expected case is disclosed as suppressed. The new secret exposure in B is not that case and Critical overrides any attempted suppression. A ratio failure already present unchanged in base belongs under relevant pre-existing issues, not as a new inline finding; an unmeasured performance concern stays an unverified risk, not a confident defect.
