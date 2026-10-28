# model-routing skill

**대부분의 작업은 Luna 6으로 처리하고, 필요한 부분만 적합한 모델에 배치하는 Coordinator 라우팅 규칙.**

Claude Code와 Codex에서 함께 쓰는 에이전트 스킬이다. 작업을 쪼개고, 각 작업에 맞는 모델(Luna 6, Terra 6.1, Sol 6.1, Sonnet 5.5, Opus 5.5, Astra)과 effort를 고르고, 실패했을 때 어디로 넘길지를 정한다. Fable·Gemini 계열은 사용하지 않는다.

- 규칙 원문: [`SKILL.md`](SKILL.md)
- 이 문서: 설치·업데이트·사용법과 규칙 요약

## 목차

1. [설치](#설치)
2. [업데이트와 제거](#업데이트와-제거)
3. [사용법](#사용법)
4. [핵심 원칙](#핵심-원칙)
5. [모델 배치](#모델-배치)
6. [effort와 비용](#effort와-비용)
7. [orchestration과 함께 사용](#orchestration과-함께-사용)
8. [수정과 기여](#수정과-기여)
9. [공식 문서](#공식-문서)

## 설치

세 방법 중 하나만 고른다. 같은 스킬을 두 방법으로 설치하면 중복으로 로드된다. 어느 쪽이든 설치 후 Claude Code·Codex 세션을 새로 열어야 스킬이 인식된다.

| 방법 | 대상 | 업데이트 | 맞는 경우 |
| --- | --- | --- | --- |
| [1. `npx skills`](#방법-1-npx-skills-권장) | Claude Code + Codex | 명령 한 줄 | 두 에이전트를 함께 쓸 때 (권장) |
| [2. Claude Code 플러그인](#방법-2-claude-code-플러그인) | Claude Code | 자동 업데이트 가능 | Claude Code만 쓸 때 |
| [3. git clone + 링크](#방법-3-git-clone--링크) | Claude Code + Codex | `git pull` | 스킬을 직접 고치며 쓸 때 |

### 방법 1. `npx skills` (권장)

Node.js 18 이상이 필요하다. 한 번에 Claude Code와 Codex 양쪽에 설치되고, 이후 명령 한 줄로 업데이트할 수 있다.

```bash
npx skills add sangwookp9591/model-routing -g -a claude-code codex -y
```

| 옵션 | 의미 |
| --- | --- |
| `-g` | 사용자 전역에 설치. 빼면 현재 프로젝트(`.claude/skills`, `.agents/skills`)에만 설치 |
| `-a claude-code codex` | 설치 대상 에이전트. 하나만 쓰면 그 에이전트만 적는다 |
| `-y` | 확인 질문 생략 |

설치 결과는 다음과 같다.

- `~/.agents/skills/model-routing`: 실제 파일 (Codex가 읽는 경로)
- `~/.claude/skills/model-routing`: 위 폴더를 가리키는 링크 (Claude Code가 읽는 경로)

설치 확인:

```bash
npx skills list -g
```

### 방법 2. Claude Code 플러그인

Claude Code 2.1.275 이상에서는 세션 안에서 한 줄로 설치한다.

```text
/plugin install model-routing --marketplace sangwookp9591/model-routing
```

이전 버전이거나 마켓플레이스를 먼저 등록하고 싶으면 두 단계로 나눈다.

```text
/plugin marketplace add sangwookp9591/model-routing
/plugin install model-routing@model-routing
```

터미널에서는 같은 동작을 `claude plugin ...`으로 실행한다.

```bash
claude plugin marketplace add sangwookp9591/model-routing
claude plugin install model-routing@model-routing
```

설치 후 `/reload-plugins`를 실행하거나 세션을 새로 연다. 플러그인으로 설치한 스킬의 명시적 호출 이름은 `/model-routing:model-routing`이다.

자동 업데이트는 직접 추가한 마켓플레이스에서 기본으로 꺼져 있다. 켜려면 `/plugin` → **Marketplaces** 탭 → `model-routing` → **Enable auto-update**를 선택한다. 켜 두면 세션을 시작할 때 새 커밋을 받아 오고, 다음 세션부터 적용된다.

### 방법 3. git clone + 링크

Node.js 없이 쓰거나, 스킬을 직접 고치면서 쓰고 싶을 때 사용한다.

```bash
git clone https://github.com/sangwookp9591/model-routing.git ~/skills/model-routing
mkdir -p ~/.claude/skills ~/.agents/skills
ln -s ~/skills/model-routing ~/.claude/skills/model-routing
ln -s ~/skills/model-routing ~/.agents/skills/model-routing
```

같은 경로에 이미 스킬이나 링크가 있으면 `ln -s`가 실패한다. 기존 항목을 확인한 뒤 필요한 것만 교체한다.

프로젝트 단위로만 쓰려면 링크 위치를 저장소 안으로 바꾼다.

| 환경 | 개인 스킬 경로 | 프로젝트 스킬 경로 | 명시적 호출 |
| --- | --- | --- | --- |
| Claude Code | `~/.claude/skills/model-routing` | `<repo>/.claude/skills/model-routing` | `/model-routing` |
| Codex | `~/.agents/skills/model-routing` | `<repo>/.agents/skills/model-routing` | `$model-routing` |

ChatGPT의 개인 스킬 생성·설치는 로컬 CLI의 폴더 연결과 별도다.

## 업데이트와 제거

| 설치 방법 | 새 버전 확인 | 업데이트 | 제거 |
| --- | --- | --- | --- |
| `npx skills` | `npx skills check` | `npx skills update model-routing` | `npx skills remove model-routing -g` |
| Claude Code 플러그인 | `/plugin` → **Installed** | `claude plugin update model-routing@model-routing` 또는 자동 업데이트 | `claude plugin uninstall model-routing@model-routing` |
| git clone | `git -C ~/skills/model-routing fetch && git -C ~/skills/model-routing status` | `git -C ~/skills/model-routing pull` | 두 링크와 clone 폴더 삭제 |

업데이트 후에는 진행 중인 세션을 새로 열어야 바뀐 규칙이 적용된다. 바뀐 내용은 저장소의 커밋 기록에서 확인한다.

## 사용법

### 호출 방식

- **자동 호출**: 모델 배치, effort 선택, 작업 분배, 승격 판단이 필요한 요청이면 에이전트가 스킬 설명을 보고 스스로 불러온다.
- **명시적 호출**: 확실하게 적용하려면 요청 앞에 붙인다. Claude Code는 `/model-routing`(플러그인 설치 시 `/model-routing:model-routing`), Codex는 `$model-routing`이다.

스킬은 모델을 **고르고 배치표를 만드는** 규칙이다. 실제로 다른 모델을 띄우려면 실행 환경에 서브에이전트·모델 선택 기능이 있어야 한다. 기능이 없으면 스킬은 배치안을 제안만 하고, 실제 실행 모델과 구분해 보고한다.

### 예시 1. 전체 리뷰 후 남은 작업 처리

```text
/model-routing
남은 작업을 먼저 정리하고, 코드 리뷰·리팩토링·기존 기능 검증을 마친 뒤 남은 작업을 처리해줘.
대부분은 Luna 6으로 진행하고, 어려운 부분만 적합한 모델에 배치해줘.
```

에이전트는 Luna로 상태를 확인해 남은 작업을 먼저 공유하고, 영역별 리뷰 → 수정 → 회귀 검증 → 남은 작업 구현 → 결과 보고 순서로 진행한다.

### 예시 2. 실행 전에 배치표만 받기

```text
/model-routing
결제 모듈 리팩토링을 작업 단위로 나눠서, 작업별 모델·effort·근거·완료 기준 표만 먼저 보여줘. 실행은 아직 하지 마.
```

응답은 다음 형식의 표로 나온다.

| 작업 | 모델 / effort | 선택 근거 | 완료 기준 |
| --- | --- | --- | --- |
| 결제 호출부 탐색·의존성 정리 | Luna 6 / low | 범위가 명확한 탐색 | 호출부 목록과 의존 그래프 |
| DTO·Mapper 분리 | Luna 6 / medium | 일반 구현 | 타입 검사·단위 테스트 통과 |
| 재시도·멱등성 로직 재설계 | Sol 6.1 / medium | 동시성·불변 조건 보존 | 동시성 테스트 통과 |
| 리팩토링 결과 독립 리뷰 | Luna 6 / medium (별도 컨텍스트) | 일반 독립 검증 | 리뷰 지적 사항 처리 |

### 예시 3. 실패한 작업을 어디로 넘길지 판단

```text
/model-routing
Luna medium으로 이 race condition을 두 번 고쳤는데 테스트가 계속 간헐적으로 실패해. 다음에 어떻게 배치할지 판단해줘.
```

스킬은 바로 상위 모델로 올리지 않는다. 재현 조건·누락된 증거·환경 문제를 먼저 확인하고, 남은 문제가 깊은 기술 분석이면 어려운 부분만 Sol에 넘긴 뒤 후속 작업은 Luna로 돌려보낸다.

### 예시 4. 화면·문서 작업

```text
/model-routing
QA 스크린샷 40장에서 오류 화면을 분류하고 주간 보고서 초안을 써줘.
```

시각 판독과 보고서 초안은 Luna로 시작하고, 판독이 부족하면 Terra medium으로 넘긴다. 코딩이 아니므로 Sol은 검토 대상이 아니다.

### 예시 5. 모델 제한을 걸어 쓰기

```text
/model-routing
이번 작업은 Claude 모델(Sonnet 5.5, Opus 5.5)만 써. 예산은 Opus를 리뷰 한 번에만 쓰는 수준으로.
```

사용자가 지정한 모델 범위·예산을 스킬 기본값보다 우선한다. 요청한 모델을 쓸 수 없으면 다른 모델로 몰래 대체하지 않고 이유를 알린다.

## 핵심 원칙

- 탐색, 계획, 구현, 일반 설계, 리팩토링, 리뷰, 테스트, 문서 작성은 Luna 6을 우선한다.
- 작업을 분해하고 완료 기준을 정한 뒤 필요한 역량에 맞춰 모델과 effort를 선택한다.
- 복잡한 부분만 전문 모델에 맡기고, 해결 후 일반 작업은 Luna로 진행한다.
- 계획·리뷰라는 이유만으로 Claude를, 브라우저·화면 조작이라는 이유만으로 Astra를 선택하지 않는다.
- 시각 판독·컴퓨터 사용·글쓰기가 핵심이고 Luna로 부족하면 Astra보다 Terra를 먼저 검토한다.
- Luna로 부족한 코딩·어려운 디버깅은 Terra를 거치지 않고 Sol에 맡긴다. Terra는 반값이지만 코딩에서는 effort를 끝까지 올려야 해서 작업당 비용이 Sol보다 커진다.
- Terra에 배치할 때는 권한·승인 설정을 끄지 않는다.
- Fable·Gemini는 승격, 대체 모델, 독립 검증에서도 제외한다.
- 모델 선택만으로 새로운 도구나 권한이 생기지는 않는다. 실행 환경의 실제 지원 여부를 확인한다.

## 모델 배치

아래는 공식 용도를 바탕으로 한 운영상 배치 예시이며, 고정 허용 목록이나 절대 성능 순위가 아니다.

| 모델 | 주요 배치 |
| --- | --- |
| **Luna 6** | 대부분의 작업의 기본 담당자. 명확한 범위의 구현·계획·리뷰·검증 |
| **Terra 6.1** | 스크린샷·문서 판독, 보고서 초안, 브라우저 작업, 양식 입력, 반복 업무 (`medium`). 코딩의 중간 단계로는 쓰지 않음 |
| **Sol 6.1** | 코딩 메인, 어려운 디버깅, 동시성, 성능, 원인 분석, 어려운 에이전트 작업 |
| **Sonnet 5.5** | 일상 개발, 빠른 반복, UI 개선, 코드 리뷰, 요구사항 해석, 문서·슬라이드 완성도 개선, 하위 에이전트 |
| **Opus 5.5** | 장시간 코딩, 큰 리팩토링·마이그레이션, 지속적인 설계 판단과 중요 리뷰 |
| **Astra** | 어려운 추론·코딩, 여러 시스템을 종합하는 분석, 복합 도구 작업 |
| **그 밖의 사용 가능한 모델** | 공식 기능·비용·실제 가용성을 확인하고 작업에 적합할 때 선택 |

OpenAI 계열은 Luna < Terra < Sol < Astra 순으로 가격과 역량이 올라간다. Astra는 복잡한 추론과 코딩에도 활용한다. Sonnet은 구현과 리뷰 모두 가능하며, Opus도 설계뿐 아니라 장시간 구현을 담당할 수 있다.

Luna로 부족할 때 다음 단계는 작업 종류로 갈린다.

| 작업 종류 | Luna 다음 단계 |
| --- | --- |
| 코딩 (구현·버그 수정·리팩토링·원인 분석) | Sol 6.1 `medium` |
| 화면·문서 (시각 판독·브라우저·양식·보고서 초안·반복 업무) | Terra 6.1 `medium` |
| 해석·디자인·리뷰의 다른 관점 | Sonnet 5.5 |
| 중요한 설계·지속적 판단, 여러 시스템 종합 | Opus 5.5 또는 Astra |

## effort와 비용

- Luna·Terra·Sol은 일반적으로 `medium`부터 시작한다. 단순 탐색·화면 조작은 `low`를 검토한다. Sol 6.1은 `none`을 지원하지 않는다.
- 코딩에서 Terra가 부족하면 effort를 올리지 말고 Sol `medium`으로 바꾼다. Terminal-Bench 4.0에서 Terra `max`는 50.3%·작업당 $1.84, Sol `medium`은 52.7%·$1.02였다.
- Sonnet 5.5는 일상 작업·하위 에이전트에서 `low`~`medium`, 어렵거나 긴 작업에서 `high`를 검토한다. Claude Code·앱 기본값은 `medium`, API 기본값은 `high`다. `high` 이상이 필요하면 Opus 5.5 `low`~`medium`과 비교한다.
- Opus 5.5는 API 기본값인 `medium`부터 시작하고 필요에 따라 조정한다.
- Astra는 이 스킬의 비용 우선 정책에 따라 `medium`부터 검토한다.
- 실제 실행 인터페이스에서 지원하는 effort만 사용한다. API의 지원 값과 앱·CLI 설정을 구분한다.
- 단가뿐 아니라 사용 토큰, 재시도, 검증, 인계 비용까지 고려한다. 상세 가격 기준일과 공식 출처는 `SKILL.md`를 참조한다.

## orchestration과 함께 사용

`orchestration`은 작업 순서·의존성·상태를 관리하고, `model-routing`은 각 작업의 모델·effort를 선택한다. 이 스킬은 orchestration 자체를 설치하거나 모델 전환 기능을 구현하지 않는다.

전체 코드 리뷰 후 남은 작업을 처리할 때는 다음 순서를 따른다.

1. 현재 상태와 남은 작업을 확인해 먼저 공유한다.
2. 전체 코드를 영역별로 리뷰한다.
3. 필요한 수정과 리팩토링을 수행한다.
4. 기존 기능과 회귀 여부를 검증한다.
5. 남은 작업을 구현하고 검증한다.
6. 완료 사항, 미검증 항목, 차단 요인을 보고한다.

일반 독립 검증은 별도 컨텍스트의 Luna를 우선한다. 전문성이 필요한 부분만 다른 모델에 맡긴다. 이미 승인·위임받은 범위는 반복 확인하지 않는다.

## 수정과 기여

- 규칙의 기준은 `SKILL.md`다. README는 요약과 사용법이므로 규칙을 바꾸면 두 파일을 함께 고친다.
- 모델·가격·벤치마크 수치를 추가할 때는 `SKILL.md` 마지막 절에 출처와 확인일을 남긴다. 공식 출처가 아니면 보조 자료로 구분한다.
- 변경은 이슈나 PR로 제안한다. `main`에 반영되면 사용자는 [업데이트](#업데이트와-제거) 명령으로 받는다.
- 플러그인 매니페스트(`.claude-plugin/plugin.json`)에는 `version`을 일부러 넣지 않는다. 비워 두면 커밋 SHA가 버전이 되어 push할 때마다 업데이트로 인식된다. `version`을 넣으면 매 배포마다 올려야 한다.
- 매니페스트를 고친 뒤에는 저장소 루트에서 `claude plugin validate .`를 실행한다.

## 라이선스

[MIT](LICENSE)

## 공식 문서

확인일: 2026-10-27. 기능·가격·설치 경로가 바뀌면 최신 공식 문서로 확인한다.

- [OpenAI 모델 가이드](https://developers.openai.com/api/docs/guides/latest-model)
- [Luna 6](https://developers.openai.com/api/docs/models/gpt-6-luna) · [Terra 6.1](https://developers.openai.com/api/docs/models/gpt-6.1-terra) · [Sol 6.1](https://developers.openai.com/api/docs/models/gpt-6.1-sol) · [Astra](https://developers.openai.com/api/docs/models/gpt-6-astra)
- [Sonnet 5.5](https://www.anthropic.com/claude-sonnet-5-5)
- [Opus 5.5](https://platform.claude.com/docs/en/models/opus-5-5/overview)
- [Claude effort](https://platform.claude.com/docs/en/build-with-claude/effort)
- [Claude Code 스킬 설치](https://code.claude.com/docs/en/skills)
- [Claude Code 플러그인 설치·업데이트](https://code.claude.com/docs/en/plugins/install)
- [Codex 스킬 설치](https://learn.chatgpt.com/docs/build-skills)
- [skills CLI (`npx skills`)](https://github.com/vercel-labs/skills)
