# 하네스 감지와 하네스별 배치 방법

2026-10-29 기준. 출처 표시: [문서]는 공식 문서, [관찰]은 실제 실행에서 확인한 값, [3자]는 다른 감지 라이브러리가 쓰는 신호다. 하네스 버전이 바뀌면 신호도 바뀔 수 있다.

## 1. 감지 순서

1. **프로필 블록:** 사용자 지침 파일(`CLAUDE.md`, `AGENTS.md`, `GEMINI.md` 등)에 `model-routing 프로필` 블록이 있으면 그것을 출발점으로 삼는다.
2. **도구 이름:** 지금 쓸 수 있는 하위 에이전트·셸 도구의 이름을 아래 2절 표와 대조한다. 가장 믿을 만한 신호다.
3. **셸 변수:** 셸에서 하네스 변수를 확인한다(3절). 하네스를 중첩 실행하면 안쪽 셸에 바깥 하네스 변수가 그대로 남는다 [관찰]. 2와 어긋나면 2를 따른다.
4. **질문:** 그래도 모호하면 사용자에게 한 번 묻는다.

판정에 쓰지 않는 신호:

- `AGENTS.md`, `.agents/skills`, `.claude/skills`, `.claude/agents`는 여러 하네스가 함께 읽는다. 이 스킬 파일이 어느 경로에서 로드됐는지도 근거가 되지 않는다.
- `AI_AGENT`는 중첩 실행에서 바깥 하네스 값이 남는다 [관찰].
- 모든 변수와 파일은 사용자가 바꿀 수 있다. 하네스 판정은 배치 힌트로만 쓰고, 승인 여부처럼 안전에 걸린 판단은 보수적으로 정한다.

## 2. 도구 이름 지문

| 하네스 | 하위 에이전트·셸 도구 | 출처 |
| --- | --- | --- |
| Claude Code | `Agent`(`subagent_type`, `model`, `effort` 인자) | [문서] `model`, [관찰] `effort` |
| Codex CLI | 하위 에이전트 생성·대기·종료 도구. 정확한 이름 확인 못 함 | [문서] |
| OpenCode | `task` + 내장 `general`·`explore` | [문서] |
| Kilo Code | `task` + 내장 `general`·`explore` (OpenCode 기반이라 도구만으로는 구분이 약함) | [문서] |
| Antigravity | `invoke_subagent`, `define_subagent`, 셸 `run_command` | [문서] |
| Cursor | Task + 내장 `Explore`·`Bash`·`Browser` | [문서] |
| GitHub Copilot CLI | 런타임 도구 이름 확인 못 함. 프로필 별칭 `agent`·`execute` | [문서] |
| Gemini CLI | 셸 `run_shell_command`, `activate_skill` | [문서] |
| Cline | `spawn_agent`, `run_commands` | [문서] |
| Amp | `Task` + `Read Thread` (플러그인 `oracle`·`librarian`·`finder`) | [문서] |
| Factory Droid | `Task` + `TaskOutput`·`TaskStop`·`GenerateDroid`, 셸 `Execute` | [문서] |
| Devin Desktop·Devin CLI (구 Windsurf) | `run_subagent`, `read_subagent` | [문서] |
| Aider | 도구 호출 없음. `/architect`, `/ask` 같은 슬래시 명령 | [문서] |

## 3. 셸 변수

| 하네스 | 변수 | 출처 |
| --- | --- | --- |
| Claude Code | `CLAUDECODE=1`, `CLAUDE_CODE_SESSION_ID`, `CLAUDE_EFFORT` | [관찰] |
| Codex CLI | `CODEX_THREAD_ID`, `CODEX_TOOL_CALL_ID`, `CODEX_SANDBOX` | [관찰] |
| Antigravity CLI | `ANTIGRAVITY_AGENT=1`, `ANTIGRAVITY_CONVERSATION_ID`, `ANTIGRAVITY_SOURCE_METADATA` | [관찰] CLI만. IDE·앱은 확인 못 함 |
| OpenCode | `OPENCODE`, `OPENCODE_PID`, `AGENT=1` | [3자] |
| Cursor | `CURSOR_AGENT` | [문서] |
| Gemini CLI | `GEMINI_CLI=1` | [문서] |
| Amp | `AMP_CURRENT_THREAD_ID`. 호환을 위해 `CLAUDECODE`도 설정한다는 보고가 있다 | [3자] |
| Cline | `CLINE_ACTIVE` | [3자] |

- 중첩 실행에서는 안쪽 하네스의 변수를 우선한다. `CODEX_TOOL_CALL_ID`, `ANTIGRAVITY_SOURCE_METADATA`처럼 도구 호출마다 바뀌는 값은 지금 호출을 실행한 하네스를 가리킨다.
- `CLAUDECODE`만으로 Claude Code라고 단정하지 않는다(Amp 가능성).

## 4. 하네스별 능력

| 하네스 | 쓸 수 있는 배치 대상 모델 | 단계별 배치 수준 | 사용량 확인 |
| --- | --- | --- | --- |
| Claude Code | Claude 계열 | 모델·effort: 에이전트 정의의 `model:`, `Agent` 호출의 `model`·`effort` | `/usage`, 상태 표시줄 |
| Codex CLI | OpenAI 계열 (`model_providers`로 다른 공급자 추가 가능) | 모델·effort: 에이전트 TOML의 `model`, `model_reasoning_effort` | `/status` |
| OpenCode | 공급자 설정에 따라 OpenAI·Claude·오픈 모델 | 모델·effort: 에이전트의 `model`(`provider/model`), `reasoningEffort` | 확인 못 함 |
| Kilo Code | 게이트웨이·BYOK로 여러 계열 | 모델·effort: `model`, 공급자 옵션 `reasoningEffort` | `kilo stats` (비용 통계) |
| Cursor | Claude 계열, Kimi·GLM·DeepSeek. GPT는 5.6 세대만 확인 | 모델·effort: `model: <id>[effort=high]` | 에디터 설정 화면 |
| Factory Droid | 확인 못 함 (BYOK는 `custom:`) | 모델·effort: `model`, `reasoningEffort` | 확인 못 함 |
| GitHub Copilot | OpenAI 계열(GPT-6·6.1)과 Claude 계열 모두, Kimi K3 | 모델만: 에이전트 프로필 `model`. effort는 세션 전역 `effortLevel` | `/usage`, 푸터 |
| Devin Desktop·Devin CLI | Astra, Sol 6.1, 오픈 모델. Claude 5.5는 확인 못 함 | 모델만: 프로필 `model` | 사용량 미터 화면 |
| Antigravity | Sonnet 5.5·Opus 5.5 (유료 Pro 이상). 4.6과 GPT-OSS-120b는 2026-11-02 제거 예정 | 지정 불가: 하위 에이전트 `model`은 `inherit`·`flash`·`pro`뿐 | 설정 화면 |
| Amp | Astra, Sol 6.1 (모드별), 외부 에이전트로 Claude Code | 지정 불가: 하위 에이전트는 스레드 모델 상속 | `amp usage` |
| Cline | 공급자 설정에 따라 OpenAI·Claude·DeepSeek·Qwen·GLM | 지정 불가: 하위 에이전트 모델 키 없음 | 확인 못 함 |
| Aider | litellm 공급자 | 하위 에이전트 없음: 메인·`--editor-model`·`--weak-model` 세 자리 | 확인 못 함 |
| Gemini CLI | 없음 (Gemini 전용) | — | — |

## 5. 단계별 배치 수준에 따른 적용

- **모델·effort 지정:** 배치표대로 단계마다 하위 에이전트를 띄운다.
- **모델만 지정:** 단계마다 모델은 배치표대로 정하고, effort는 세션 설정 하나로 맞춘다. 배치표 effort와 다른 단계는 배치 공유에 표시한다.
- **지정 불가:** 세션 모델 하나로 진행한다. 단계별 배치표는 제안으로 공유하고, 등급이 크게 다른 단계(T0와 T2 이상)가 섞이면 세션 모델 전환을 제안한다. Amp는 모드(`low`·`medium`·`high`) 전환으로 근사한다.
- **하위 에이전트 없음(Aider):** `--weak-model`에 T0, `--editor-model`에 T1, 메인(architect) 모델에 T2를 둔다.
- **배치 대상 모델 없음:** Gemini CLI, 그리고 2026-11-02 이후 Free·Plus 플랜의 Antigravity가 해당한다. 배치 대상 모델이 없다고 알리고 배치표는 제안으로만 공유한다. 실제 실행 모델과 구분하고, 다른 모델로 자동 대체하지 않는다.
- Cursor는 지정한 모델을 쓸 수 없으면 호환 모델로 조용히 바꾼다 [문서]. 하위 에이전트에게 실제 사용한 모델을 결과에 적게 한다.
- Claude Code의 `haiku` 별칭은 Bedrock·Google Cloud·Foundry에서 Haiku 4.5를 가리킨다. `claude-haiku-5-5`로 고정한다.

## 6. 승인 상태 확인 (Terra 규칙)

GPT-6.1 Terra를 쓸 수 있는 하네스는 Codex, OpenCode, GitHub Copilot이다(Cursor·Amp는 GPT-5.6 Terra만 확인). 다음 중 하나라도 해당하면 "승인 불가"로 보고 Terra를 배치하지 않는다. 확인할 수 없어도 "승인 불가"로 본다.

| 하네스 | 승인 불가 신호 |
| --- | --- |
| Codex CLI | `approval_policy = "never"`, `sandbox_mode = "danger-full-access"`, `--dangerously-bypass-approvals-and-sandbox`, 헤드리스 `codex exec`. 하위 에이전트는 부모의 승인 설정을 상속한다. 승인 정책은 셸 변수로 보이지 않으므로 `~/.codex/config.toml`·`.codex/config.toml`과 실행 플래그를 확인하거나 사용자에게 묻는다 |
| OpenCode | `"permission": "allow"` 또는 `bash`·`edit`가 `allow`, 헤드리스 `opencode run` |
| GitHub Copilot | `-p` 실행에 `--allow-all`·`--yolo`, `COPILOT_ALLOW_ALL`, autopilot 모드, cloud agent |
| Cline (BYOK로 Terra를 쓸 때) | CLI는 `--auto-approve` 기본값이 true. `spawn_agent`를 승인하면 하위 에이전트의 모든 동작이 승인된다 |
| Amp | 기본적으로 승인 없이 도구를 실행한다는 보고가 있다 [2차] |

공통: 사람이 승인할 수 없는 헤드리스 실행(`claude -p`, `codex exec`, `opencode run`, `agy -p`, `copilot -p`, `kilo run`, `droid exec`, `cursor -p`)에는 Terra를 넘기지 않는다.

## 7. 현재 모델·effort 확인

- Claude Code: 시스템 프롬프트의 모델 문구, 셸 `CLAUDE_EFFORT` [관찰].
- Codex: `~/.codex/config.toml`의 `model`·`model_reasoning_effort` (세션 중 `/model` 전환은 반영되지 않을 수 있음).
- OpenCode·Kilo: `opencode.json`·`kilo.jsonc`의 `model`.
- Copilot: `COPILOT_MODEL`, `~/.copilot/settings.json`의 `model`·`effortLevel`. 커스텀 에이전트의 `model`이 우선한다.
- 그 밖에는 사용자에게 묻거나 "미상"으로 둔다. 모델의 자기 보고만으로 단정하지 않는다.
