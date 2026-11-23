# 독립 Codex 스킬 직접 설치

현재 번들: [`critical-code-review/`](critical-code-review/SKILL.md). Claude 없이 Codex에서 코드/diff/PR을 리뷰하며, Claude→Codex `codex-loop`의 JSON 결과 파이프라인과는 별도다. 전체 폴더(`SKILL.md`, `agents/`, `references/`)를 복사한다.

## 직접 설치 (Claude 불필요)

1. 이 저장소를 별도 작업 폴더에 clone하고 기본 `main`의 원하는 커밋을 확인한다.
2. `critical-code-review/` 전체를 사용자 또는 프로젝트 스킬 폴더에 복사한다. 기존 동명 설치가 있으면 경로/내용을 확인하고 교체 여부를 결정한다. 여러 위치에 중복 설치하지 않는다.
3. Codex CLI/IDE의 `/skills` 또는 `$` 선택기에서 `critical-code-review`와 로드된 경로를 확인한다. 반영되지 않으면 Codex를 재시작한다.

저장소 루트에서 실행하는 예시 (기존 대상이 있으면 덮어쓰지 않고 중단):

```powershell
$reviewSource = Join-Path (Get-Location) 'codex-review/skills/setup-codex-review/codex-skills/critical-code-review'
$reviewSkills = Join-Path $HOME '.agents/skills'
$reviewTarget = Join-Path $reviewSkills 'critical-code-review'
if (Test-Path -LiteralPath $reviewTarget) { throw "기존 설치를 먼저 확인하세요: $reviewTarget" }
New-Item -ItemType Directory -Force -Path $reviewSkills | Out-Null
Copy-Item -LiteralPath $reviewSource -Destination $reviewTarget -Recurse
```

macOS/Linux에서는 저장소 루트에서:

```sh
review_source="codex-review/skills/setup-codex-review/codex-skills/critical-code-review"
review_skills="$HOME/.agents/skills"
if [ -e "$review_skills/critical-code-review" ]; then
  echo "기존 critical-code-review 설치를 먼저 확인하세요"
else
  mkdir -p "$review_skills" && cp -R "$review_source" "$review_skills/"
fi
```

[현재 Codex 로컬 스킬 안내](https://developers.openai.com/codex/skills/)의 사용자 경로는 `~/.agents/skills/`, 프로젝트 범위는 `<repo>/.agents/skills/`다. 프로젝트 범위 설치 시 위 대상만 변경한다. 기존 Claude 셋업 명령 `/codex-review:setup-codex-review`는 `~/.codex/skills/`에 복사하는 별도 경로를 유지하므로, 사용 중인 Codex의 실제 로드 경로를 확인해 동명 사본을 중복 선택하지 않는다.

## 호출·검증

Codex 대화에서 (아래 저장소/번호는 예시):

- `$critical-code-review 현재 local diff를 리뷰해줘. 게시하지 말고 심각도와 merge blocker/follow-up, 근거와 최소 수정안을 보여줘.`
- `$critical-code-review OWNER/REPO PR 123을 리뷰해줘.` → findings와 게시용 초안만 제공.
- `$critical-code-review OWNER/REPO PR 123을 리뷰하고 확정된 중요한 문제는 변경 줄에 댓글 달아줘.` → 해당 PR 게시 권한·검토 head·diff 좌표를 확인한 뒤 COMMENT inline note.
- 상세 근거나 구조 검토가 필요하면 “상세 리뷰” / “구조 심층 검토”를 덧붙인다.

설치 확인은 작은 local diff로 먼저 한다. 대상/head·요구사항 대조·단일 심각도·처리 구분·검증 한계가 나오고, 게시 요청이 없으면 댓글을 쓰지 않아야 한다. 실제 PR에 테스트 댓글을 게시하지 않는다. GitHub 직접 게시의 정확한 필드/좌표/중복·실패 처리는 [inline 운영 절차](critical-code-review/references/github-inline.md)를 따른다. Orca PR 이벤트로 깨어난 봇은 지정된 `orca bot reply --note ...` 경로만 쓴다.

## 배포 참고

정책을 변경할 때는 [합성 dry-run 사례](critical-code-review/references/validation-cases.md)로 좌표·권한·stale head·근거 부족·clean PR 판단을 확인한다. 이 사례 통과는 실제 GitHub 게시나 로컬 설치 검증을 대신하지 않는다.

Claude 사용자는 기존 `/codex-review:setup-codex-review`로 번들 복사를 할 수도 있다. 이 폴더에는 공개 재배포 권한이 있는 스킬만 넣고, 회사 주소·계정·비밀값은 넣지 않는다. 독립 스킬은 JSON loop 스키마를 요구하지 않는다.
