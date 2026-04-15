# `execute.py` 생성을 위한 마스터 프롬프트

## Role
너는 복잡한 개발 작업을 여러 단계(Step)로 나누어 자가 교정(Self-correction)하며 수행하는 "AI 워크플로우 오케스트레이터"를 설계하는 시니어 데브옵스 엔지니어다.

## Goal
Python 표준 라이브러리만 사용하여, 특정 Phase 내의 Step들을 순차적으로 실행하고 관리하는 `scripts/execute.py`를 작성하라.

## Core Logic (Business Rules)
1. **상태 관리**: `phases/{phase}/index.json` 파일을 읽어 각 step의 `status`를 관리한다. (`pending`, `completed`, `error`, `blocked`)
2. **프롬프트 조립 (Preamble)**:
   - 가드레일: `CLAUDE.md`와 `docs/*.md` 내용을 모든 요청 앞에 주입한다.
   - 컨텍스트: 이전 step들에서 성공한 `summary` 필드를 모아 "이전 작업 결과"로 제공한다.
   - 에러 피드백: 실패 후 재시도 시, 이전 시도의 에러 메시지를 프롬프트에 포함하여 자가 수정을 유도한다.
3. **LLM 호출**: `claude -p --output-format json` 커맨드를 subprocess로 실행하여 AI 모델을 호출한다. (timeout 30분)
4. **결과 처리**:
   - 성공 시: `status`를 `completed`로 바꾸고 타임스탬프와 AI가 작성한 `summary`를 기록한다.
   - 실패 시: 최대 3회 재시도한다. 3회 초과 시 `error` 상태로 기록하고 종료한다.
   - 차단 시: AI가 사용자 개입을 요청하면 `blocked` 상태로 기록하고 즉시 중단한다.
5. **Git 자동화**:
   - 실행 전 `feat-{phase}` 브랜치를 생성하거나 체크아웃한다.
   - 각 step 완료 후, 실제 코드 변경사항(`feat`)과 메타데이터/인덱스 업데이트(`chore`)를 나누어 2단계로 커밋한다.

## Technical Requirements
- Python 3.8+ 표준 라이브러리만 사용 (외부 pip 설치 없이 동작해야 함).
- 터미널에서 경과 시간을 보여주는 애니메이션 Spinner(◐◓◑◒)를 스레드로 구현할 것.
- 모든 시간 기록은 KST(Asia/Seoul, +09:00) 기준 ISO 8601 형식을 사용할 것.
- JSON 저장 시 `ensure_ascii=False`와 `indent=2`를 적용하여 가독성을 확보할 것.

## Deliverables
1. `scripts/execute.py`: 위 로직이 구현된 실행 파일.
2. `scripts/test_execute.py`: `pytest`를 사용하여 주요 로직(JSON 처리, 프롬프트 조립, 시간 계산, Mock 기반 Git 호출)을 검증하는 테스트 코드.
