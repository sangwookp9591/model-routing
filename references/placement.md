# 작업 유형별 배치

## 배치표

근거 강도: **높음**은 하네스 차이를 감안해도 독립 측정 두 곳 이상이 같은 방향인 경우, **중간**은 독립 측정이 하나이거나 독립 소표본 실험인 경우, **낮음**은 제조사 발표만 있거나 해당 작업의 직접 측정이 없는 경우다.

| 작업 유형 | Codex (OpenAI) | Claude | 승격·예외 | 근거 | 강도 |
| --- | --- | --- | --- | --- | --- |
| 탐색·분류: 파일·심볼 검색, 로그·테스트 출력 분류, 빌드·테스트·정해진 시나리오 실행 | Luna / low | Haiku 5.5 / low | 결과를 믿기 어려우면 Terra low 또는 Haiku medium. 도구 호출 자체는 승격 사유가 아니다 | [독립] AA v5.0 Luna low 10점·$0.01, Terra low 29점·$0.07. [독립] 소표본 실험에서 기계적 위임 작업은 Haiku로 충분 | 중간 |
| 계획·남은 작업 정리 | Terra / medium | Sonnet 5.5 / medium | 중요한 설계 판단은 Sol high 또는 Opus high | [독립] AA v5.0 종합 지수만 있고 계획 전용 측정은 없음 | 낮음 |
| 일반 구현: 범위가 정해진 CRUD, 단일 이슈 버그 수정, 테스트 작성 | Terra / medium, high까지. 단일 파일 기계적 편집만 Luna medium | Haiku 5.5 / medium, 테스트·빌드 실행을 지시 | 의존성·복잡한 로직이 남으면 Sol medium. Haiku가 두 번 실패하면 Sonnet medium | [독립] AA TB4 Haiku 33% 대 Luna 13~16.4%. AA v5.0 Terra medium 38점·$0.17 > Luna max 28점·$0.15. [2차] Terra TB4 55.6% | 중간 (Terra 코딩은 낮음) |
| 어려운 코딩·디버깅: 의존적 다중 파일, 가설 검증형 디버깅, 동시성, 긴 터미널 작업 | Sol 6.1 / medium, 막히면 high | Opus 5.5 / high | 저장소 단위 수정·마이그레이션은 Astra xhigh. Opus max는 쓰지 않는다. Sonnet은 xhigh가 상한이고 Opus high보다 점수가 낮고 비싸다 | [독립] Harbor TB4 Sol 58.2%·$0.6k, Astra 58.2%·$3.3k, Opus 64.8%·$4.7k. DeepSWE Astra xhigh 74%. [제조사] Opus high 64.2%·$3.88 대 max 64.8%, Sonnet xhigh 61.5%·$5.30 | 중간~높음 |
| 코드 리뷰·요구사항 누락 확인 | Terra / medium | Sonnet 5.5 / medium | 중요한 변경은 Sol medium, 설계 쟁점은 Opus high. 미묘한 리뷰·보안 리뷰에 Haiku를 쓰지 않는다 | [독립] 소표본 실사용 보고에서 미묘한 리뷰·보안은 Sonnet으로 올림. 리뷰 전용 측정 없음 | 낮음 |
| 화면 조작: 읽기 위주, 부작용 없음 | Terra / medium, 승인 설정 유지 | Haiku 5.5 / medium | 실패하면 Sol medium (Codex) 또는 Sonnet high (Claude) | [제조사] OSWorld 2.1 Haiku 72.4%(strict 37.1%), Terra medium 66.7% | 낮음 |
| 화면 조작: 다단계·권한 경계(로그인, 결제, 삭제, 양식 제출) | Sol 6.1 / medium. Terra 금지(SKILL.md §2-7) | Sonnet 5.5 / high | Opus 5.5 | [제조사] OSWorld 2.1 Sonnet 83.9%(strict 48.8%), Sol max 78.0%, Terra 차단 우회 29.6%. [독립] arena Agent Opus 1위, Sonnet 2위(약한 근거) | 낮음 |
| 시각 판독: 스크린샷, 차트, 도면, 문서 이미지 | Terra / medium | Opus 5.5 / high. 단순 판독은 Haiku medium | 판독 오류가 남으면 Sol high (Codex) | [제조사] MMMU-Pro Terra 84.6~85.5%, Astra 86.7%. [독립] arena Vision Sol 1294 ≈ Opus 1293 > Sonnet 1265 > Luna 1240, Terra·Haiku 미등재 | 낮음 |
| 보고서·글쓰기: 사람에게 나가는 문서 | 초안 Sol 6.1 / medium | 초안 Sonnet 5.5 / high | 최종본은 Opus high. 저가 초안은 Terra medium(제조사 근거만). Haiku·Luna는 쓰지 않는다 | [독립] arena Text Opus 1509, Sol 1480 ≈ Sonnet 1478 ≈ Astra 1476, Haiku 1425. GDPval-AA Opus 1846 | 중간 |
| 사실 민감한 요약·추출 | Terra / medium, 원문 인용 강제 | Haiku 5.5 / medium, 주어진 문서에서 추출할 때만 | 오류 비용이 크면 Sol high. Luna는 쓰지 않는다 | [독립] AA-Omniscience 환각률 Terra 38%, Haiku 40%, Luna 77%. [제조사] 사실 오류 Terra medium 10.1%, max 4.3%, Sol max 4.4% | 중간 |
| 장시간 연구·여러 시스템 종합 | Astra / high | Opus 5.5 / high | xhigh는 평가 후. 저가 1차 수집은 Terra medium(독립 미검증) | [독립] ALE v1.1 Opus·Astra 공동 1위 42.6%. AA v5.0 Opus high 57점·$6.05, Astra high 55점·$3.66 | 중간 |
| 하위 에이전트 | 검증 가능한 반환은 Luna low~Terra low | 검증 가능한 반환은 Haiku 5.5 / low~medium, 에이전트 정의에 `model: claude-haiku-5-5` | 판단이 들어가는 글, 보안 리뷰, 의존적 다중 파일 편집, 100k 토큰이 넘는 루프는 Terra medium (Codex) 또는 Sonnet medium (Claude) | [독립] 소표본 실험에서 에이전트 정의 18개 중 14개를 Haiku로 옮김. [제조사] Anthropic 모델 개요 | 중간 |

Codex에서 일반 구현은 Terra medium에서 시작해 high까지 쓴다. 그 이상이 필요하면 effort를 올리지 말고 Sol medium으로 바꾼다.

## 단계 분해 예시

**버그 수정**

| 단계 | 결과 확인 방법 | Codex | Claude | 이유 |
| --- | --- | --- | --- | --- |
| 1. 로그·스택 수집과 분류 | 원본 로그로 다시 확인 | Luna / low | Haiku 5.5 / low | 검증 가능한 기계 작업 |
| 2. 오류 화면 스크린샷 판독 | 원인 분석 단계에서 화면과 대조 | Terra / medium | Haiku 5.5 / medium, 어긋나면 Opus high | 시각 판독. 근거 낮음 |
| 3. 원인 분석·가설 검증 | 확인 없이 다음 단계의 판단 근거가 됨 | Sol 6.1 / medium | Opus 5.5 / high | 어려운 디버깅 |
| 4. 수정·회귀 테스트 작성 | 테스트·빌드 | 범위가 좁으면 Terra medium, 다중 파일이면 Sol medium | 범위가 좁으면 Haiku medium, 다중 파일이면 Opus high | 원인이 정해지면 수정은 테스트로 검증 가능 |
| 5. 독립 리뷰 | 별도 컨텍스트에 요구사항·diff·테스트 로그 제공 | Terra / medium | Sonnet 5.5 / medium | 구현과 다른 모델의 관점 |
| 6. 결과 보고 | 사람에게 바로 나감 | 3단계 모델이 이어서 작성 | 3단계 모델이 이어서 작성 | 짧은 보고는 인계 비용이 더 큼 |

**관리 화면 대량 등록**

| 단계 | 결과 확인 방법 | Codex | Claude | 이유 |
| --- | --- | --- | --- | --- |
| 1. 입력 데이터 정리·검증 | 스크립트 검사 | Luna / low | Haiku 5.5 / low | 검증 가능한 기계 작업 |
| 2. 화면 구조 파악·필드 매핑 | 1건 시험 등록으로 확인 | Terra / medium, 승인 설정 유지 | Haiku 5.5 / medium | 읽기 위주 화면 조작 |
| 3. 등록 제출 반복 | 건별 승인 또는 시험 계정 | Sol 6.1 / medium | Sonnet 5.5 / high | 권한 경계. Terra 금지 |
| 4. 결과 대조 | 스크립트로 목록 비교 | Luna / low | Haiku 5.5 / low | 검증 가능한 기계 작업 |

## 근거가 낮은 배치의 확정과 재검토

- 근거 강도가 '낮음'인 배치는 실제 과제 3~5건을 1순위 모델과 승격 열 모델로 각각 실행해 같은 기준(테스트 통과, 판독 정답, 사람 채점)으로 채점한다. 작업당 비용과 시간도 기록한다.
- 1순위 모델의 품질이 승격 모델과 같으면 1순위를 확정하고, 낮으면 승격 모델을 그 작업의 기본값으로 바꾼다. 결과는 배치 공유(SKILL.md §8)에 함께 적는다.
- 다음 경우에 이 표를 다시 검토한다. Terra·Haiku가 Harbor Terminal-Bench, Agents' Last Exam, arena Text·Vision 같은 독립 리더보드에 등재될 때, 대상 모델의 새 버전이 나올 때, Artificial Analysis 지수 버전이 바뀔 때. 지수 버전이 다른 점수끼리는 비교하지 않는다.
