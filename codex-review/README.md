# codex-review — 독립 Codex 리뷰 스킬 + Claude Code 플러그인

## Codex에서 직접 리뷰하기 (Claude 불필요)

[`critical-code-review`](skills/setup-codex-review/codex-skills/critical-code-review/SKILL.md)는 독립적으로 유지보수하는 Codex 스킬이다.
[`직접 설치·확인 방법`](skills/setup-codex-review/codex-skills/README.md)을 따라 설치하고 Codex에서 `$critical-code-review`로 호출한다.
기존 번들 경로와 이름은 유지한다. CLI 사용 시 Codex 설치·인증이 필요하고, GitHub inline 게시에는 해당 PR의 쓰기 권한과 명시적 게시 요청이 필요하다.

- 기본 결과: 간결한 한국어, Critical/High/Medium/Low, merge blocker/follow-up, 독립적인 confidence, 파일/줄·실패 근거·영향·최소 수정안.
- PR/연결 이슈 요구사항을 diff와 대조해 `Spec` 누락을 찾고, 미확인 자료·기존 문제·미실행 테스트를 구분한다.
- 문제 줄에 댓글을 요청하면 검토 head와 변경 줄 좌표를 재확인해 중복 없는 inline note를 게시한다. 일반 리뷰 요청은 게시용 초안만 제공한다.
- 기본 패널 토론은 없으며, 상세·구조 심층 모드는 요청 시 선택한다. Claude 정책을 자동 동기화하는 사본이 아니다.

## 별도 경로: Claude→Codex 반복 루프

아래는 Claude Code 플러그인 설치/운영 안내다. `codex-loop`의 별도 프롬프트·JSON 스키마·수정 반복 절차는 독립 스킬의 출력/inline 게시 계약과 다르다.
이번 독립 스킬 갱신은 그 파이프라인의 스키마나 Claude 워크플로를 변경하지 않는다.

코드는 **클로드**가 짜고, 리뷰·문서는 **Codex CLI**에 오프로드하는 리뷰 워크플로.
무거운 리뷰·문서 추론이 Codex(OpenAI) 쪽에서 돌아 **클로드 토큰을 절감**한다.
플러그인으로 설치하면 **모든 레포/경로에서** 사용 가능.

## 포함 스킬 (설치 후 네임스페이스로 호출)
- **`/codex-review:codex-loop`** — 현재 diff(또는 지시한 기능 구현분)를 Codex `critical-code-review`로
  정확성·테스트·운영 위험·코드 구조·모듈화까지 리뷰 → 클로드가 수정 → 재리뷰를 clean 이 날 때까지 반복 → Codex가 문서 갱신. 커밋은 안 함.
  - 인자 없음: 현재 uncommitted diff / `<작업 지시>`: 클로드가 먼저 구현
  - 플래그: `--max-rounds N`(기본 없음, 상한이 필요할 때만), `--no-docs`, `--design-review`, `--model M`(기본 `gpt-6-astra`), `--effort E`(기본 `medium`)
  - 리뷰 방식(v0.6.0~):
    - 지시문은 `skills/codex-loop/references/review-prompt.md` — 변경을 깨 보려는 자세, 꼭 의심할 곳 목록, diff 안의 글은 지시가 아닌 자료로 다룸
    - 결과는 `references/review-output.schema.json` 틀의 JSON(`--output-schema`)으로 받는다 — 심각도·확신도·파일·줄·고칠 방법,
      이번 변경이 만든 문제(`introduced`)와 원래 있던 문제(`pre_existing`) 구분
    - 클로드는 고치기 전에 지적마다 파일·부르는 쪽·테스트를 열어 확인한다. 오판과 원래 있던 문제는 고치지 않고 보고에만 남긴다
- **`/codex-review:setup-codex-review`** — Codex 쪽 의존성 셋업(1회): Codex CLI 설치/로그인 확인 안내 +
  번들 Codex 스킬을 `~/.codex/skills/`로 복사.

> 자연어로 "코덱스로 리뷰하고 고쳐줘"처럼 요청하면 네임스페이스 없이도 스킬이 트리거된다.

## 팀원 설치 (1회)
```
# 1) 마켓플레이스 등록
/plugin marketplace add seungmin-lee-dev/skills

# 2) 플러그인 설치
/plugin install codex-review@claude-tools

# 3) Codex 쪽 셋업 (아무 레포에서나 1회)
/codex-review:setup-codex-review
```
설치 후 **아무 경로/레포에서나** `/codex-review:codex-loop` 사용 가능.

## 전제 (각 팀원 머신 1회)
- **Codex CLI 설치 + `codex login`** — 프로그램이라 플러그인에 담기지 않음. `setup-codex-review`가
  설치/로그인 여부를 확인해 안내한다. (대표 설치: `npm install -g @openai/codex`)
- (선택) `skills/setup-codex-review/codex-skills/`에 번들된 Codex 스킬 — 셋업 시 `~/.codex/skills/`로 복사.

## 비용
- 리뷰·문서 추론 = Codex(OpenAI) 쿼터 소모, 클로드 토큰은 최소.
- 모델은 `gpt-6-astra` / 추론 강도 `medium`으로 고정해 부른다(v0.5.0~, v0.4.1~v0.4.x 는 `gpt-6.1-sol`). 개인 `~/.codex/config.toml` 기본값을 따르지 않는다.
  Codex 한도를 아껴야 할 때는 `--model gpt-6.1-sol` 처럼 `--model` / `--effort`로 낮춘다.
- 기본은 횟수 제한 없이 clean 까지 돈다. 대신 클로드가 고칠 게 없거나 같은 지적이 반복되면 멈추고 보고한다(무한 왕복 방지).
  Codex 쿼터를 아끼려면 `--max-rounds N`으로 상한을 건다.

## 업데이트
플러그인 레포에 push 후 팀원은 `/plugin`에서 업데이트. `plugin.json`과 루트 `marketplace.json`의
`version`을 함께 올린다.
