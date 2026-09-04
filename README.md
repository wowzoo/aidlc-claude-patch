# AI-DLC v2 — Claude Code 배포본 (patch 적용판)

순정 AI-DLC v2 **2.7.1** 배포본(`dist/claude`)에 우리 patch 를 얹은 트리다. **plugin 은 얹지 않았다.**

| | |
|---|---|
| 대상 harness | Claude Code |
| 순정 버전 | **2.7.1** (`aidlc-version.ts` 의 `AIDLC_VERSION` 실측) |
| 순정 base 커밋 | **`a277af21`** — `awslabs/aidlc-workflows` 의 `main` |
| 파일 수 | **280** = 순정 277 + 우리 추가 3 |
| spec 형상 | `remove 0` · `patch 9파일` · `add 3파일` |
| 층 구성 | 하나 (patch) |
| plugin | **미적용** (`visual-mockups` · `code-map` 둘 다 없다) |
| 복사 시점 | 2026-09-04 KST |

## 이 patch 가 무엇을 하는가

세 갈래다.

**1. 순정이 강제하는 실행 환경을 걷어낸다 (N축).** 순정 `.claude/settings.json` 은 provider·region·model 을
못박는다. Claude Code 에서는 프로젝트 settings 가 user settings 를 outrank 하므로, 그대로 쓰면 **이 배포본을
놓은 것만으로 쓰는 사람 자신의 설정이 덮인다.** 같은 기계에서 `/status` 를 두 번 찍어 확인한 이동이다
(us-west-2 → us-east-1, opus-5 → opus-4-8). 이 patch 는 그 강제를 제거하는 것뿐이고, 대신 값을 박지 않는다.

**2. 엔진 결함 4건을 고친다.** 실런을 실제로 멈춘 것들이고, 넷 다 `doctor` 와 `graph compile --check` 를
통과한다 — 즉 순정 검증만으로는 잡히지 않는다.

**3. 우리 자산 3건을 싣는다.**

## 패치 목록 — 2.7.1 기준 스냅샷

⚠️ **이것은 2.7.1 한 세대의 스냅샷이다.** 살아 있는 정본은 워크스페이스의
`.claude/skills/aidlc-claude-patch/register.md` 이고, 세대마다 전 항목을 다시 잰다. 좌표(줄 번호)는 **순정**
기준이므로 이 트리에서 그대로 찾으면 어긋난다.

### 순정 강제 제거 — N축 (3건)

| ID | 무엇 | 대상 파일 |
|---|---|---|
| N-1 | `env` 의 provider·region·model 핀과 `"model"`·`"effortLevel"` 을 제거한다 | `.claude/settings.json` |
| N-2 | `.example` 이 그 핀을 되살리는 것을 막는다 | `.claude/settings.local.json.example` |
| N-4 | 그 강제를 기정사실로 서술하는 산문을 고친다 | `.claude/CLAUDE.md` |

### 엔진 결함 정정 — `.ts` 코드 (4건)

| ID | 무엇 | 대상 파일 |
|---|---|---|
| D-8 | 정렬 안 된 audit row 가 run floor 를 어긋내 **유닛 완료 영수증을 전량 폐기**하는 것을 고친다 | `.claude/tools/aidlc-lib.ts` |
| D-9 | 엔진이 스스로 발행한 `load-steering` directive 를 처리하는 동안 **계획서 쓰기까지 거절**되는 것을 고친다 | `.claude/hooks/aidlc-plan-approval-guard.ts` |
| D-10 | 턴 경계마다 규칙 전달이 part 1 로 되감기고, 그 재발행이 **Plan Approval 증거를 삭제**하는 것을 고친다 (marker 에 전달된 bundle digest 를 커서로 둔다) | `.claude/tools/aidlc-orchestrate.ts` · `.claude/tools/aidlc-lib.ts` |
| D-11 | 숫자 승인(`1`/`2`)이 기록 전에 파괴돼 **엔진이 스스로 제공하는 단축키가 도달 불가**인 것을 고친다 | `.claude/hooks/aidlc-record-human-turn.ts` |

> `D-11` 을 조금 더 풀면: 순정은 사람의 답을 먼저 `JSON.parse` 하는데 `"1"` 은 유효 JSON 이라 숫자 `1` 이
> 되고, 문자열도 객체도 아니므로 빈 문자열로 지워진다. 그러면 Plan Approval 응답이 아예 기록되지 않아
> **번호로 답하면 승인이 성립하지 않는다.** 이 patch 를 얹으면 번호가 통하고, 얹지 않은 순정에서는
> 제시된 라벨(`Approve Plan`)을 그대로 입력해야만 넘어간다.

### 순정 산문 결함 정정 (3건)

| ID | 무엇 | 대상 파일 |
|---|---|---|
| D-5 | 실물과 맞지 않는 hook 계수 서술 | `.claude/CLAUDE.md` |
| D-6 | 구현이 없는 worktree merge dispatch 를 서술하는 고아 산문 | `knowledge/aidlc-shared/audit-format.md` · `knowledge/aidlc-pipeline-deploy-agent/branching-strategies.md` |
| D-7 | heading 앞 빈 줄 결손 2곳 | `.claude/CLAUDE.md` |

### 우리 자산 (3건)

| ID | 무엇 | 대상 |
|---|---|---|
| L-1 | `aidlc-git-merge` 스킬 (3파일) | `.claude/skills/aidlc-git-merge/` |
| G-2 | `permissions.allow` 에 `WebFetch` 한 줄 — 순정은 `WebSearch` 만 준다 | `.claude/settings.json` |
| F-8 | `## Working Language` 절 | `.claude/CLAUDE.md` 의 마지막 섹션 |

### 일부러 손대지 않은 것

| ID | 무엇 | 이유 |
|---|---|---|
| N-3 | `.mcp.json` 서버 구성 | 순정 그대로 5서버로 둔다 — 이 층의 범위 밖 |
| G-1 | `permissions.deny` 신설 | 만들지 않기로 결정했다 |
| D-3 | 트리에 없는 `docs/` 를 인용하는 산문 | 결함이지만 이 세대에 고치지 않는다(HOLD) |

## 쓰는 법

**전제** — `bun` 이 `PATH` 에 있어야 한다. 엔진 도구 전부가 `bun <tool>.ts` 로 돈다.

**설치** — 이 디렉터리의 내용을 프로젝트 루트에 그대로 둔다. 네 항목이다.

```
.claude/      엔진 (tools · hooks · agents · skills · knowledge · settings)
aidlc/        워크플로 데이터가 쌓이는 셸 (씨앗만 들어 있다)
.mcp.json     MCP 서버 등록 5개 (순정 그대로)
.gitignore
```

**진입점** — Claude Code 에서 `/aidlc <하고 싶은 것>`.

## 검증

```bash
bun .claude/tools/aidlc-utility.ts doctor
```

**50 passed, 0 failed** 이 나온다.

🔴 판정 기준은 이 절대값이 아니라 **순정과 같은가** 다. 같은 base 의 순정 트리도 50 이므로, 우리 델타가
검사 항목을 늘리지도 줄이지도 않았다는 것이 그 등식으로 확인된다. 상류가 검사를 더하면 숫자는 그날
움직이니, 다음 세대에는 순정을 다시 재서 대조한다.

⚠️ **`doctor` 가 묻지 않는 것이 있다** — hook 등록 · `statusLine` · MCP 활성화 · `env` · `@`-import 체인.
그리고 우리 델타가 트리에 실제로 들어 있는지도 묻지 않는다. 그래서 전수 검증은 워크스페이스의
`aidlc-claude-patch` 스킬 배터리(`scripts/verify.py`)가 맡는다 — 2026-09-04 실측 **PASS**(sha 전수
byte-exact · doctor 50-0 · `graph compile --check` exit 0 · `loadAgents` 14 · hook 참조 MISSING 0).

## 다시 세우려면

이 트리는 손으로 만든 것이 아니라 기계로 도출된 것이다. 워크스페이스
(`~/Development/aidlc-v2-patch`)에서 `aidlc-claude-patch` 스킬을 쓴다.

```bash
C=.claude/skills/aidlc-claude-patch
R=~/Development/ai-dlc/_single-source-of-truth/aidlc-workflows-v2
git -C "$R" archive a277af21 dist/claude | tar -x -C /tmp/stock-2.7.1-claude
uv run python $C/scripts/build.py  --stock /tmp/stock-2.7.1-claude/dist/claude --out /tmp/claude-2.7.1
uv run python $C/scripts/verify.py --tree  /tmp/claude-2.7.1
```

## 주의

- **plugin 은 별도 작업이다.** `visual-mockups`(시각 mockup) 와 `code-map`(외부 코드맵) 은 이 트리에 없다.
  얹으려면 워크스페이스의 `plugins/README.md` 절차를 따른다. 얹으면 MCP 서버가 5 → 7 로 늘고
  `.aidlc-plugin/` 표시가 생기므로, 적용 여부는 그 둘로 판별한다.
- **이 트리를 고쳐 쓰지 말 것.** 고칠 것이 생기면 워크스페이스의 트리를 고치고 `respec.py` 를 돌린 뒤
  다시 복사한다. 사본을 직접 고치면 두 벌이 갈리고, 스펙은 그 갈림을 잡아 주지 않는다.
- 실런 데이터가 쌓이기 시작하면 `aidlc/` 아래가 그 프로젝트의 것이 된다. 다음 세대로 올릴 때 `aidlc/` 를
  덮어쓰지 않도록 주의한다.
