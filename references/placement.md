# 작업 유형별 배치

배치는 두 표를 차례로 쓴다. 등급표로 작업의 등급과 필요 능력을 정하고, 배치표에서 그 행의 모델을 고른다. 작업 유형과 모델을 분리해 두면 하네스나 모델이 늘어도 등급표는 그대로 두고 배치표만 고치면 된다.

## 등급

- **T0 기계:** 결과를 테스트·스크립트·상위 모델이 다시 확인할 수 있는 좁은 단계.
- **T1 일반:** 범위가 정해져 있고 결과를 테스트나 다음 단계에서 확인할 수 있는 단계.
- **T2 어려움:** 결과가 확인 없이 사람·커밋·배포로 나가거나, 실패 비용이 크거나, 원인 추적처럼 추론이 깊은 단계.
- **T3 최상위:** 저장소 단위 수정, 장시간 연구, 여러 시스템을 종합하는 단계.

등급을 정하는 규칙:

- 결과를 확인할 수 있으면 T0·T1에서 시작한다. 확인 없이 밖으로 나가면 한 등급 올린다.
- 로그인·결제·삭제처럼 권한 경계가 있으면 T2 이상이고 Terra를 배치하지 않는다(SKILL.md §2-7).
- 작업이 길거나 파일이 많다는 이유만으로 등급을 올리지 않는다.

## 등급표: 작업 유형 → 등급·능력

| 작업 유형 | 등급·능력 | 등급을 바꾸는 조건 |
| --- | --- | --- |
| 파일·심볼 검색, 로그·테스트 출력 분류, 빌드·테스트·정해진 시나리오 실행 | T0 기계 | 도구 호출 자체는 승격 사유가 아니다 |
| 계획·남은 작업 정리 | T1 판단 | 중요한 설계 판단은 T2 판단 |
| 범위가 정해진 구현: CRUD, 단일 이슈 버그 수정, 테스트 작성 | T1 코딩 | 의존성·복잡한 로직이 남으면 T2 코딩 |
| 어려운 코딩·디버깅: 의존적 다중 파일, 가설 검증형 디버깅, 동시성, 긴 터미널 작업 | T2 코딩 | 저장소 단위 수정·마이그레이션은 T3 |
| 코드 리뷰·요구사항 누락 확인 | T1 판단 | 중요한 변경·설계 쟁점은 T2 판단 |
| 화면 조작: 읽기 위주, 부작용 없음 | T1 화면 | 실패하면 T2 화면 |
| 화면 조작: 다단계·권한 경계(로그인, 결제, 삭제, 양식 제출) | T2 화면·권한 경계 | — |
| 시각 판독: 스크린샷, 차트, 도면, 문서 이미지 | T1 시각 | 판독 오류가 남으면 정밀 판독 |
| 보고서·글쓰기: 사람에게 나가는 문서 | T2 글쓰기 | 최종본은 T3 글쓰기 |
| 사실 민감한 요약·추출 | T1 사실 | 오류 비용이 크면 T2 사실 |
| 장시간 연구·여러 시스템 종합 | T3 연구 | — |

하위 에이전트에 맡기는 단계도 같은 표로 정한다. 조회·요약·분류·로그처럼 검증 가능한 반환은 T0, 판단이 들어가는 글, 보안 리뷰, 의존적 다중 파일 편집, 100k 토큰이 넘는 루프는 T1 판단 이상이다.

## 배치표: 등급·능력 → 모델 × effort

열은 하네스가 아니라 모델 계열이다. 하네스에서 쓸 수 있는 계열의 열을 쓰고, 두 계열을 모두 쓸 수 있으면 근거 열의 [독립] 수치로 고르되 계열을 넘는 인계 비용을 함께 따진다.

근거 강도: **높음**은 하네스 차이를 감안해도 독립 측정 두 곳 이상이 같은 방향인 경우, **중간**은 독립 측정이 하나이거나 독립 소표본 실험인 경우, **낮음**은 제조사 발표만 있거나 해당 작업의 직접 측정이 없는 경우다.

| 등급·능력 | OpenAI 계열 | Claude 계열 | 승격·예외 | 근거 | 강도 |
| --- | --- | --- | --- | --- | --- |
| T0 기계 | Luna / low | Haiku 5.5 / low | 결과를 믿기 어려우면 Terra low 또는 Haiku medium | [독립] AA v5.0 Luna low 10점·$0.01, Terra low 29점·$0.07. [독립] 소표본 실험에서 기계적 위임 작업은 Haiku로 충분 | 중간 |
| T1 코딩 | Terra / medium, high까지. 단일 파일 기계적 편집만 Luna medium | Haiku 5.5 / medium, 테스트·빌드 실행을 지시 | T2 코딩으로. Haiku가 두 번 실패하면 Sonnet medium | [독립] AA TB4 Haiku 33% 대 Luna 13~16.4%. AA v5.0 Terra medium 38점·$0.17 > Luna max 28점·$0.15. [2차] Terra TB4 55.6% | 중간 (Terra 코딩은 낮음) |
| T1 판단 | Terra / medium | Sonnet 5.5 / medium | 중요한 변경은 Sol medium. 미묘한 리뷰·보안 리뷰에 Haiku를 쓰지 않는다 | [독립] AA v5.0 종합 지수만 있음. 소표본 실사용 보고에서 미묘한 리뷰·보안은 Sonnet으로 올림. 계획·리뷰 전용 측정 없음 | 낮음 |
| T1 화면 | Terra / medium, 승인 설정 유지 | Haiku 5.5 / medium | 실패하면 T2 화면 | [제조사] OSWorld 2.1 Haiku 72.4%(strict 37.1%), Terra medium 66.7% | 낮음 |
| T1 시각 | Terra / medium | 단순 판독은 Haiku 5.5 / medium, 정밀 판독은 Opus 5.5 / high | 판독 오류가 남으면 Sol high | [제조사] MMMU-Pro Terra 84.6~85.5%, GPT-6 Astra 86.7%. [독립] arena Vision Opus 1295 ≈ Sol 6.1 1293 > Sonnet 1263. Sol 6.2·Astra 6.2·Terra·Haiku 미등재 | 낮음 |
| T1 사실 | Terra / medium, 원문 인용 강제 | Haiku 5.5 / medium, 주어진 문서에서 추출할 때만 | T2 사실은 Sol high (Claude 계열은 Sonnet high). Luna는 쓰지 않는다 | [독립] AA-Omniscience 환각률 Terra 38%, Haiku 40%, Luna 77%. [제조사] 사실 오류 Terra medium 10.1%, max 4.3%, Sol 6.1 max 4.4%. Sol 6.2의 수치는 없음 | 중간 |
| T2 코딩 | Sol 6.2 / medium, 막히면 high | Opus 5.5 / high | T3로. Sol high로 부족하면 Astra 6.2 medium과 비교. Opus max는 쓰지 않는다. Sonnet은 xhigh가 상한이고 Opus high보다 점수가 낮고 비싸다 | [독립] Vals TB4 Sol 6.2 60.6%·$1.13, Astra 6.2 63.1%·$4.69, Opus 65.2%·$12.04. AA 지수 Sol 6.2 medium 53점, high 55점. Harbor에는 6.2 미등재(Sol 6.1 60.9%·$0.6k, Opus 65.8%·$4.7k). [2차] AA 코딩 에이전트 지수는 Sol 6.1보다 3점 낮음. [제조사] Opus high 64.2%·$3.88 대 max 64.8%, Sonnet xhigh 61.5%·$5.30 | 중간~높음 (Sol 6.2는 중간) |
| T2 판단 | Sol 6.2 / high | Opus 5.5 / high | — | [독립] AA 지수 Sol 6.2 high 55점(작업당 비용 미공개, max $1.01), Opus high 52점·$3.46. 설계 판단 전용 측정 없음 | 낮음 |
| T2 화면·권한 경계 | Sol 6.2 / medium. Terra 금지 | Sonnet 5.5 / high | Opus 5.5 | [제조사] OSWorld 2.1 Sonnet 83.9%(strict 48.8%), Sol 6.1 max 78.0%(Sol 6.2 수치 없음), Terra 차단 우회 29.6%. 에이전트 메시지에 따른 무단 행동 80회 중 Sol 6.2 0회, Terra 11회. [독립] arena Agent Opus 1위, Sonnet 2위(약한 근거) | 낮음 |
| T2 글쓰기 (초안) | Sol 6.2 / medium | Sonnet 5.5 / high | 저가 초안은 Terra medium(제조사 근거만). Haiku·Luna는 쓰지 않는다 | [독립] arena Text Opus 1507, Sol 6.1 1481 ≈ GPT-6 Astra 1474 ≈ Sonnet 1473, Haiku 1425. Sol 6.2 미등재 | 중간 (Sol 6.2는 낮음) |
| T3 글쓰기 (최종본) | Opus 5.5 / high (쓸 수 없으면 Sol 6.2 high) | Opus 5.5 / high | — | [독립] arena Text Opus 1507, GDPval-AA Opus 1846 | 중간 |
| T3 코딩 (저장소 단위) | Astra 6.2 / xhigh | Opus 5.5 / high | Opus xhigh는 평가 후. Astra max는 쓰지 않는다 | [독립] Vals TB4 Astra 6.2 63.1%·$4.69 (GPT-6 Astra 59.6%·$9.58, Opus 65.2%·$12.04). AA TB4 Astra 6.2 max 64.1%. AA 지수 Astra 6.2 xhigh 62점 = max 62점. DeepSWE는 GPT-6 Astra xhigh 74%만 있음 | 중간 |
| T3 연구 | Astra 6.2 / high | Opus 5.5 / high | xhigh는 평가 후. 저가 1차 수집은 Terra medium(독립 미검증) | [독립] ALE v1.1 Opus 43.4%, GPT-6 Astra 42.6%(Astra 6.2 미등재). AA 지수 Opus high 52점·$3.46, Astra 6.2 high 60점(작업당 비용 미공개, max $4.00) | 중간 |

- OpenAI 계열 T1 코딩은 Terra medium에서 시작해 high까지 쓴다. 그 이상이 필요하면 effort를 올리지 말고 Sol 6.2 medium으로 바꾼다.
- Terra를 배치할 수 없는 환경(승인 불가, SKILL.md §2-7)의 T1은 기계적 편집이면 Luna medium, 그 밖에는 Sol 6.2 / low로 대신한다. AA 지수에서 Sol 6.2 low는 41점·작업당 1.4분으로 Terra medium(38점·2.9분)과 Terra high(42점) 사이다 [독립]. 작업당 비용은 공개되지 않았고 토큰 단가는 Terra의 2배다. 근거 강도는 낮음이다.
- Sol 6.2·Astra 6.2를 쓸 수 없는 하네스에서는 Sol·Astra 자리에 Sol 6.1·GPT-6 Astra를 같은 effort로 쓴다. 근거 수치는 `references/models.md`의 이전 세대 행을 본다.
- Claude 계열 하위 에이전트에 Haiku를 쓰려면 에이전트 정의에 `model: claude-haiku-5-5`를 적는다.

## 오픈 모델 대안

배치표의 1순위 계열을 쓸 수 없거나 그 계열의 예산이 부족할 때, 또는 하네스·과금상 오픈 모델이 실제 선택지일 때만 쓴다. 모델 정보와 운영 주의는 `references/models.md`의 오픈 모델 절에 있다.

| 등급·능력 | 오픈 모델 | 쓰는 조건 | 근거 | 강도 |
| --- | --- | --- | --- | --- |
| T1 코딩 | GLM-5.3-Flash | Haiku·Terra를 쓸 수 없거나 그쪽 예산이 부족할 때 | [독립] Terminal-Bench 4.0 Snorkel 35.8%, Vals.ai 37.4%·$1.22 (Haiku 5.5 36.4%·$3.12). DeepSWE 63%·$0.24 | 중간 |
| T1 코딩 (프런트엔드 초안) | DeepSeek V4.1 Flash 1017 | 저가 초안. 미국·유럽 호스팅 제공사 우선 | [독립] arena WebDev 1665 ≈ Terra max 1673. 에이전트 코딩 독립 측정 없음 | 낮음 |
| T1 시각 (정밀 판독) | Qwen3.8 Max | 다른 계열의 정밀 판독을 쓸 수 없을 때. 작업당 비용이 높다 | [독립] arena Vision 1300(2위) > Sol 6.1 1294 ≈ Opus 1293 (2026-10-29 값) | 중간 |
| T2 코딩 (긴 터미널 작업) | Kimi K3.1 Pro / max | Opus를 쓸 수 없거나 Claude 계열 예산이 부족할 때. Sol 6.2를 쓸 수 있으면 Sol이 더 싸고, Astra 6.2는 점수와 비용이 비슷하다 | [독립] Terminal-Bench 4.0 Vals.ai 63.6%·$4.57, Artificial Analysis 67.2% (Opus 5.5 65.2%·$12.04, Astra 6.2 63.1%·$4.69, Sol 6.2 60.6%·$1.13) | 중간~높음 |

- T0는 오픈 모델을 쓰지 않는다. Luna low·Haiku low가 더 싸다.
- 보안 리뷰·보안 작업에는 오픈 모델을 배치하지 않는다.
- 회사 코드나 민감한 데이터는 사용자 확인 없이 중국 회사의 공식 API로 보내지 않는다.
- Qwen3.8 Max(코딩)와 MiniMax-M3.1-Flash는 종합 지수에서 다른 모델에 지배되므로 배치하지 않는다.

## 단계 분해 예시

**버그 수정**

| 단계 | 결과 확인 방법 | 등급·능력 | OpenAI 계열 | Claude 계열 |
| --- | --- | --- | --- | --- |
| 1. 로그·스택 수집과 분류 | 원본 로그로 다시 확인 | T0 기계 | Luna / low | Haiku 5.5 / low |
| 2. 오류 화면 스크린샷 판독 | 원인 분석 단계에서 화면과 대조 | T1 시각 | Terra / medium | Haiku 5.5 / medium, 어긋나면 Opus high |
| 3. 원인 분석·가설 검증 | 확인 없이 다음 단계의 판단 근거가 됨 | T2 코딩 | Sol 6.2 / medium | Opus 5.5 / high |
| 4. 수정·회귀 테스트 작성 | 테스트·빌드 | 범위가 좁으면 T1 코딩, 다중 파일이면 T2 코딩 | Terra medium 또는 Sol medium | Haiku medium 또는 Opus high |
| 5. 독립 리뷰 | 별도 컨텍스트에 요구사항·diff·테스트 로그 제공 | T1 판단 | Terra / medium | Sonnet 5.5 / medium |
| 6. 결과 보고 | 사람에게 바로 나감 | 3단계 모델이 이어서 작성 | — | — |

짧은 보고는 따로 배치하면 인계 비용이 더 크다.

**관리 화면 대량 등록**

| 단계 | 결과 확인 방법 | 등급·능력 | OpenAI 계열 | Claude 계열 |
| --- | --- | --- | --- | --- |
| 1. 입력 데이터 정리·검증 | 스크립트 검사 | T0 기계 | Luna / low | Haiku 5.5 / low |
| 2. 화면 구조 파악·필드 매핑 | 1건 시험 등록으로 확인 | T1 화면 | Terra / medium, 승인 설정 유지 | Haiku 5.5 / medium |
| 3. 등록 제출 반복 | 건별 승인 또는 시험 계정 | T2 화면·권한 경계 | Sol 6.2 / medium | Sonnet 5.5 / high |
| 4. 결과 대조 | 스크립트로 목록 비교 | T0 기계 | Luna / low | Haiku 5.5 / low |

## 근거가 낮은 배치의 확정과 재검토

- 근거 강도가 '낮음'인 배치는 실제 과제 3~5건을 1순위 모델과 승격 열 모델로 각각 실행해 같은 기준(테스트 통과, 판독 정답, 사람 채점)으로 채점한다. 작업당 비용과 시간도 기록한다.
- 1순위 모델의 품질이 승격 모델과 같으면 1순위를 확정하고, 낮으면 승격 모델을 그 작업의 기본값으로 바꾼다. 결과는 배치 공유(SKILL.md §8)에 함께 적는다.
- 다음 경우에 이 표를 다시 검토한다. Sol 6.2·Astra 6.2가 Harbor Terminal-Bench, DeepSWE, Agents' Last Exam, arena에 등재되거나 Artificial Analysis가 effort별 작업당 비용을 공개할 때, Terra·Haiku가 Harbor Terminal-Bench, Agents' Last Exam, arena Text·Vision 같은 독립 리더보드에 등재될 때, 대상 모델의 새 버전이 나올 때, Artificial Analysis 지수 버전이 바뀔 때. 지수 버전이 다른 점수끼리는 비교하지 않는다.
