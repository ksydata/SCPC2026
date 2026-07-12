**진행 업데이트:** 핵심 파일 검토 완료 — 프로젝트 요약과 제출 체크리스트를 작성했습니다.

**프로젝트 개요**
- **목적:** 공모전 예선 제출용 작품 준비 (SCPC2026 Final).  
- **핵심 산출물:** 제출은 SCPC 포맷의 JSON/CSV (schema: `scpc.final.answer.v1`) 형태로 제출합니다. 참고 파일:
  - TERMS_GUIDE.md — 데이터 구조·용어 안내
  - submission_schema.json — 제출 JSON 스키마
  - sample_submission.csv — 제출 CSV 예시
  - dev_tasks.jsonl — 공개 개발용 task 샘플

**구성 요소 (간단)**
- 데이터
  - `data/dev_tasks.jsonl` (JSONL) — 연습용 120개 task
  - `data/dev_answers.json` (JSON) — 공개 정답(검증용)
  - `data/screening_tasks.jsonl` — 예선용/스크리닝 task
- 실행/예시
  - SCPC2026_Final_baseline.ipynb — 데이터 로드, baseline harness, 제출파일 생성 예시
  - sample_submission.csv — baseline 이름과 빈 answers 구조 예시
- 제출 규칙 요점 (스키마 요약)
  - 최상위 `schema` 필수: "scpc.final.answer.v1"
  - `answers` 객체: 각 `task_id` 별로 객체 존재
  - 각 답안 객체 필수 필드: `focal_id`, `target`, `control`(proceed/amend/hold/ask), `content_scope`(mode: raw/summary/redacted/status_only/none 등), `policy`, `plan_events`(최대 18), 기타 선택적 필드(`user_response`, `audit_tags`, `counterfactual`)
  - `meta`에 harness 정보(이름, 외부 API 사용 여부, model_id, temperature 등) 기록 권장

**현재 상태 요약 (검토한 파일 기준)**
- TERMS_GUIDE.md: task 구조·용어와 제출 예시, plan_events 의미·추천 args bucket 등 상세 가이드 포함.
- submission_schema.json: 제출 형식과 필드 제약(필수/열거형/배열 길이 등)을 명확히 정의.
- sample_submission.csv: baseline 빈 제출 예시(answers 비어 있음) — 실제 제출 전 answers 채워야 함.
- dev_tasks.jsonl: 다양한 도메인(메시지/일정/파일/health 등)과 안전·공유 정책 신호가 포함된 복합 task 샘플 다수.

**예선 제출을 위한 최소 체크리스트**
- **스키마 검증:** submission_schema.json에 따라 생성한 JSON이 유효한지 로컬에서 JSON Schema 검증.
- **answers 키 매칭:** 각 task id(예: `task_...`)를 `answers`의 key로 사용하고, 필수 필드 모두 포함.
- **`plan_events` 길이 제한:** 항목 수 최대 18개 준수.
- **열거형 값 확인:** `control`, `content_scope.mode` 등 허용값(스키마)을 사용.
- **meta 입력:** `harness_name`, `uses_external_api` 등 기본 메타값 기입.
- **샘플 실행:** SCPC2026_Final_baseline.ipynb 노트북으로 데이터 로드 → 제출파일 생성 흐름 시연.
- **검증 데이터로 테스트:** `data/dev_answers.json`(로컬 채점용 레퍼런스)이용해 포맷/매핑 확인.
- **최종 제출 파일:** sample_submission.csv 형식(또는 대회 안내 형식)에 맞춰 내보내기.

**권장 즉시 작업(우선순위)**
- 1) 노트북 실행: SCPC2026_Final_baseline.ipynb를 열어 baseline 흐름 확인.
- 2) 답안 생성 스크립트(harness) 구현: 각 task에 대해 `answers[task_id]` 객체를 만드는 최소 파이프라인 구현.
- 3) 스키마 자동검증 추가: 제출 전 JSON Schema로 자동 검증(실패 시 로그 출력).
- 4) 샘플 테스트: dev_tasks.jsonl로 파이프라인 돌려 제출 샘플 생성 및 `data/dev_answers.json`과 구조 비교.
- 5) 제출물 정리: 최종 `submission.csv` / `submission.json` 및 간단 README(제작한 harness 실행법)를 준비.

**빠른 한 페이지(제출용) — 복사해 제출요약으로 사용 가능**
- 프로젝트: SCPC2026 Final 예선 제출 준비  
- 핵심 산출물: 제출 JSON (`scpc.final.answer.v1`) / 제출 CSV(대회 안내 형식)  
- 사용한 파일: TERMS_GUIDE.md, submission_schema.json, sample_submission.csv  
- 준비상태: 데이터·스키마 확인 완료, baseline 노트북과 dev task 샘플 존재. 필요한 작업: harness 구현 → 스키마 검증 → 샘플 제출 생성 → 제출물 패키징.  
- 즉시 액션(권장): 노트북 실행 → 간단한 답안 생성 스크립트 1개 작성(예: 모든 task에 `control: "ask"` 형태로 임시 생성) → 스키마 검증 통과 확인 → 제출 샘플 생성.

원하시면 지금 바로:
- SCPC2026_Final_baseline.ipynb 실행용 간단 커맨드/절차를 만들어 드리거나,
- 예선 제출용 최소 `harness.py`(task → answers 생성 + 스키마 검증) 샘플을 생성해 드리겠습니다.


---


## 해야 할 것: sample_submission.csv의 `answers` 채우기

현재 제출 파일 상태:
```json
"answers": {}   ← 비어있음. 이걸 채워야 함
```

---

## 절차

### 1단계 — task ID 목록 파악
`screening_tasks.jsonl`에 있는 각 task의 `id` 값을 전부 수집합니다.

```bash
python3 -c "
import json
with open('SCPC2026_Final_data/data/screening_tasks.jsonl') as f:
    ids = [json.loads(l)['id'] for l in f]
print(f'총 {len(ids)}개')
print(ids[:3])
"
```

### 2단계 — 각 task 에 답안 객체 하나씩 만들기

`answers` 안에 `task_id: {필수필드}` 형태로 넣어야 합니다.

```python
answer_template = {
    "focal_id": "",               # ← task의 objects 중 핵심 대상 obj id
    "target": "",                 # ← 최종 전달 대상 (수신처 이름)
    "control": "proceed",         # ← proceed / amend / hold / ask 중 1개
    "content_scope": {
        "mode": "summary",        # ← raw / summary / redacted / status_only / none
        "allowed_fields": [],
        "excluded_fields": [],
        "requires_user_confirmation": False
    },
    "policy": {
        "risk_flags": [],
        "violations": [],
        "requires_confirmation": False
    },
    "plan_events": [],            # ← 최대 18개
    "user_response": "",          # ← 선택 (사용자에게 보여줄 응답)
    "audit_tags": []              # ← 선택
}
```

### 3단계 — 제출 JSON 만들기

```python
import json

tasks = []
with open('SCPC2026_Final_data/data/screening_tasks.jsonl') as f:
    for line in f:
        tasks.append(json.loads(line))

answers = {}
for task in tasks:
    tid = task['id']
    answers[tid] = {
        "focal_id": task['device_state']['objects'][0]['id'],  # 일단 첫번째
        "target": "",
        "control": "ask",
        "content_scope": {
            "mode": "summary",
            "allowed_fields": [],
            "excluded_fields": [],
            "requires_user_confirmation": False
        },
        "policy": {
            "risk_flags": [],
            "violations": [],
            "requires_confirmation": False
        },
        "plan_events": []
    }

submission = {
    "schema": "scpc.final.answer.v1",
    "meta": {
        "harness_name": "my_harness",
        "uses_external_api": False,
        "fixed_slm_policy": "local_fixed_slm_only",
        "model_id": "scpc-final-fixed-slm-local-facade",
        "temperature": 0.0,
        "seed": 42
    },
    "answers": answers
}

# CSV 형식으로 저장
import csv
with open('Sol/submission.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(['submission'])
    writer.writerow([json.dumps(submission, ensure_ascii=False)])

print("완료")
```

### 4단계 — `dev_tasks`로 먼저 테스트

`screening_tasks` 대신 dev_tasks.jsonl로 먼저 돌려보고 `dev_answers.json`과 비교해서 구조가 맞는지 확인합니다.

---

## 요약 (뭘 바꿔야 하나)

| 뭘 바꿔야 하나 | 어디서 |
|---|---|
| `focal_id` | task의 `device_state.objects` 중 prompt에 해당하는 obj의 `id` |
| `target` | task의 `device_state.records` 중 `resolved_target`의 `value` |
| `control` | 상황에 따라 `proceed / amend / hold / ask` |
| `plan_events` | 처리 순서 (read → verify → dispatch 등) |

