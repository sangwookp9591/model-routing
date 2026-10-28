# model-routing skill

**대부분의 작업은 Luna 6으로 처리하고, 필요한 부분만 적합한 모델에 배치하는 Coordinator 라우팅 규칙.**

Fable·Gemini 계열은 사용하지 않는다. 나머지 모델은 실제 가용성과 작업 적합성을 확인해 활용한다.

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

## 로컬 CLI 설치

`SKILL.md`가 있는 이 폴더를 Claude Code와 Codex의 스킬 경로에 연결하면 같은 내용을 공유할 수 있다. 다음 명령은 이 폴더에서 실행한다.

```bash
routing_skill_dir="$(pwd -P)"
test -f "$routing_skill_dir/SKILL.md" || exit 1
mkdir -p "$HOME/.claude/skills" "$HOME/.agents/skills"
ln -s "$routing_skill_dir" "$HOME/.claude/skills/model-routing"
ln -s "$routing_skill_dir" "$HOME/.agents/skills/model-routing"
```

같은 경로가 이미 있으면 링크 생성은 실패한다. 기존 스킬이나 링크를 확인한 뒤 필요한 항목만 교체한다.

| 환경 | 개인 스킬 경로 | 프로젝트 스킬 경로 | 명시적 호출 |
| --- | --- | --- | --- |
| Claude Code | `~/.claude/skills/model-routing` | `<repo>/.claude/skills/model-routing` | `/model-routing` |
| Codex | `~/.agents/skills/model-routing` | `<repo>/.agents/skills/model-routing` | `$model-routing` |

프로젝트 단위로만 사용하려면 표의 프로젝트 경로 아래에 동일한 폴더를 연결한다. ChatGPT의 개인 스킬 생성·설치와 로컬 CLI의 폴더 연결은 별도다.

## 사용 예시

Claude Code:

```text
/model-routing
남은 작업을 먼저 정리하고, 코드 리뷰·리팩토링·기존 기능 검증을 마친 뒤 남은 작업을 처리해줘.
대부분은 Luna 6으로 진행하고, 어려운 부분만 적합한 모델에 배치해줘.
Fable과 Gemini는 사용하지 마.
```

Codex에서는 위 요청의 `/model-routing`을 `$model-routing`으로 바꿔 호출한다.

## 공식 문서

확인일: 2026-10-27. 기능·가격·설치 경로가 바뀌면 최신 공식 문서로 확인한다.

- [OpenAI 모델 가이드](https://developers.openai.com/api/docs/guides/latest-model)
- [Luna 6](https://developers.openai.com/api/docs/models/gpt-6-luna) · [Terra 6.1](https://developers.openai.com/api/docs/models/gpt-6.1-terra) · [Sol 6.1](https://developers.openai.com/api/docs/models/gpt-6.1-sol) · [Astra](https://developers.openai.com/api/docs/models/gpt-6-astra)
- [Sonnet 5.5](https://www.anthropic.com/claude-sonnet-5-5)
- [Opus 5.5](https://platform.claude.com/docs/en/models/opus-5-5/overview)
- [Claude effort](https://platform.claude.com/docs/en/build-with-claude/effort)
- [Claude Code 스킬 설치](https://code.claude.com/docs/en/skills)
- [Codex 스킬 설치](https://learn.chatgpt.com/docs/build-skills)
