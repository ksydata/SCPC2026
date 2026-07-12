[예선 주제] 
AI Agent Harness 설계를 통한 창의적 문제 해결



 [문제 설명]
참가자는 주어진 과제 데이터를 바탕으로 개인 기기 agent가 마주치는 요청 맥락을 이해하고, 필요한 판단과 실행 계획을 수립하여 정해진 형식의 답안을 생성하는 AI Agent Harness를 설계·구현합니다.

본 과제의 핵심은 단순히 하나의 답을 맞히는 것이 아니라, 동일한 공개 데이터와 동일한 제출 형식 안에서 task state, visible history, session memory, 정책·안전 신호를 얼마나 일관되게 해석하는지에 있습니다. 따라서 평가는 모델 크기나 외부 API 성능 차이가 아니라, 참가자가 설계한 agent logic이 주어진 맥락을 얼마나 정확하고 신뢰성 있게 처리하는지를 중심으로 이루어집니다.

좋은 GPU나 외부 대형 모델을 많이 쓰는 것이 본질인 대회가 아닙니다. 좋은 성능은 주어진 fixed SLM interface와 task JSON을 바탕으로 parser, session memory, safety/control logic, content scope, plan construction을 얼마나 잘 설계했는지에서 나와야 합니다.

참가자는 단순한 키워드 분류기나 특정 공개 예시에만 맞춘 lookup table이 아니라, 새로운 task stream에도 적용 가능한 작은 agent harness를 만드는 것을 목표로 해야 합니다. 좋은 harness는 보통 다음 질문을 순서대로 처리합니다.

현재 요청에서 중심이 되는 object는 무엇인가?
최종 동작의 대상 또는 수신처는 어디인가?
그대로 진행할지, 범위를 줄여 진행할지, 보류할지, 사용자 확인이 필요한지 어떻게 판단할 것인가?
민감 정보, 동의,
판단 결과를 구조화된 content_scope, policy, plan_events로 어떻게 표현할 것인가?
 보안 알림, 세션 이력, 사용자 메모리를 어떻게 함께 반영할 것인가?
예선 단계에서 참가자는 제공된 Public Screening 과제에 대해 로컬 환경에서 답안을 생성하고, 생성된 answer JSON을 DACON 제출 형식에 맞춰 submission.csv의 단일 셀에 담아 제출합니다. 제출된 CSV는 서버에서 JSON으로 복원된 뒤, 서버 보관 정답과 비교되어 점수가 산출됩니다.
입출력 형식, 데이터 구성, 제출 예시 및 용어집은 대회 데이터 페이지를 통해 제공됩니다.



[배포용 데이터 구조]
대회 데이터 페이지에는 다음 파일이 제공됩니다.

screening_tasks.jsonl: 예선 평가 대상 700개 문제
dev_tasks.jsonl: 연습용 문제
dev_answers.json: 일부 연습용 dev 문제에 대한 참조 답안 예시
sample_submission.csv: 제출 CSV 예시
submission_schema.json: 제출 JSON 구조
TERMS_GUIDE.md: task와 제출 JSON의 주요 용어 설명
SCPC2026_Final_baseline.ipynb: 참가자용 Python 통합 실행 예시
처음 시작하는 참가자는 TERMS_GUIDE.md로 task와 answer JSON의 필드를 확인한 뒤, SCPC2026_Final_baseline.ipynb를 실행해 전체 흐름을 보는 것을 권장합니다. 이후 baseline의 FinalHarness 내부 함수들을 하나씩 개선하면 됩니다.



[Baseline] SCPC AI Agent Harness 기반 답안 생성 및 제출
안녕하세요. 데이콘입니다.
본 게시글에서는 SCPC 2026 AI Agent Harness 과제의 베이스라인 코드를 제공합니다.
본 베이스라인은 주어진 dev_tasks.jsonl, screening_tasks.jsonl 데이터를 불러오고, 예시 AnswerHarness 구조를 통해 과제별 답안을 생성한 뒤, DACON 제출 형식에 맞는 submission.csv 파일을 생성합니다.

참가자는 본 베이스라인을 참고하여 Harness의 계획 수립, 도구 사용, 응답 생성, 검증 로직 등을 자유롭게 개선할 수 있습니다. 단, 최종 제출 파일은 반드시 submission 컬럼 1개로 구성된 CSV 형식이어야 하며, 해당 셀 안에는 전체 답안 JSON이 문자열 형태로 포함되어야 합니다.
본 베이스라인에 포함된 채점 관련 로직은 참가자의 로컬 테스트와 제출 형식 이해를 돕기 위한 예시이며, 실제 리더보드 점수는 서버에 보관된 비공개 정답 기준으로 산출됩니다.

감사합니다.
데이콘 드림



[3. Harness 작성 영역]
참가자는 보통 이 영역을 가장 많이 수정합니다. `FinalHarness.answer_task(task, session)`은 task 하나를 받아 answer JSON 하나를 반환합니다.

`session`은 같은 실행 stream 안에서 유지되는 dict입니다. 이전 turn에서 얻은 정보를 이후 turn에 활용해야 하는 유형을 다룰 때 사용할 수 있습니다.

권장 구조는 다음과 같습니다.

- `update_session_memory`: 현재 task에서 이후에 참고할 정보를 저장합니다.
- `choose_focal`: 중심 object를 고릅니다.
- `infer_target`: 최종 대상, 수신처, 앱, 채널, 장치, 메모리 저장소를 정합니다.
- `decide_control`: `proceed`, `amend`, `hold`, `ask`를 정합니다.
- `build_content_scope`: 사용할 정보와 제외할 정보를 정합니다.
- `build_policy`: 위험 신호와 확인 필요 여부를 정리합니다.
- `build_plan_events`: 처리 계획을 action 목록으로 만듭니다.

아래 코드는 일부러 약하게 작성된 starter입니다. 각 함수의 TODO 주석이 참가자가 개선할 지점입니다.
- `plan_events[*].args`는 공개 ontology의 의미 bucket을 사용해 각 단계의 근거를 표시합니다. 특정 문자열을 외워 맞히는 것이 아니라, record/scope/policy 신호에서 필요한 근거를 구조화하는 연습으로 보세요.


https://journalblog.tistory.com/1