# Harness 프레임워크 및 클로드 동작 방식 설명

이 프로젝트는 **Harness(하네스) 프레임워크**를 통해 클로드가 체계적이고 자가 교정(Self-correction)이 가능한 방식으로 작업을 수행하도록 설계되었습니다.

---

## 1. 클로드의 Harness 동작 방식 (CLAUDE.md ~ 실행)

클로드는 프로젝트의 루트에 있는 `CLAUDE.md`를 시작점으로 삼아 프로젝트의 성격과 규칙을 파악하며, 이후 `.claude/commands/harness.md`에 정의된 **워크플로우**에 따라 동작합니다.

### A. 진입 및 가이드라인 파악 (`CLAUDE.md`)
- `CLAUDE.md`에는 프로젝트의 기술 스택, 핵심 아키텍처 규칙(CRITICAL), 개발 프로세스 및 기본 명령어가 정의되어 있습니다.
- 클로드는 이를 통해 "무엇을 지켜야 하는지"와 "어떤 명령어를 사용할 수 있는지"를 가장 먼저 이해합니다.

### B. 워크플로우 단계
1.  **탐색 (Exploration):** `/docs/` 하위의 설계 문서들을 읽고 기획 의도를 파악합니다.
2.  **논의 (Discussion):** 구체적인 구현 방안이나 기술적 결정을 사용자에게 제안하고 확정합니다.
3.  **Step 설계 (Planning):** 작업을 원자적(Atomic)인 여러 단계(Step)로 쪼개어 `phases/` 디렉토리 구조를 설계합니다. 각 Step은 독립된 세션에서 실행될 수 있도록 "자기완결성"을 갖춰야 합니다.
4.  **파일 생성 (Setup):** `phases/index.json` (전체 현황) 및 `phases/{task-name}/` 내부에 전용 `index.json`과 각 단계별 가이드인 `stepN.md` 파일들을 생성합니다.
5.  **실행 (Execution):** 전용 스크립트(`execute.py`)를 통해 클로드가 각 단계를 순차적으로 수행하도록 합니다.

---

## 2. 문서(docs)를 읽어들이는 시점

문서들은 작업의 정확도와 일관성을 유지하기 위해 여러 시점에 걸쳐 반복적으로 읽힙니다.

1.  **초기 탐색 단계:** 새로운 태스크를 시작할 때 클로드가 전체적인 맥락을 잡기 위해 `/docs/` 내부의 PRD, ARCHITECTURE, ADR 등을 읽습니다.
2.  **매 Step 실행 시 (Guardrails):** `scripts/execute.py`가 실행될 때, `_load_guardrails()` 함수를 통해 `/CLAUDE.md`와 `/docs/*.md` 전체 내용을 자동으로 읽어들여 클로드의 프롬프트 서두에 주입합니다. 즉, **모든 개별 작업 단계마다 최신 설계 문서를 컨텍스트로 보유**하게 됩니다.
3.  **리뷰 단계 (Review):** `.claude/commands/review.md` 명령이 수행될 때, 변경 사항이 아키텍처나 ADR에 정의된 규칙을 준수하는지 검증하기 위해 해당 문서들을 다시 확인합니다.

---

## 3. 주요 명령어 및 역할

### A. Harness 실행 스크립트 (`scripts/execute.py`)
이 스크립트는 클로드를 "조종"하는 자동화 엔진 역할을 합니다.
-   `python3 scripts/execute.py {task-name}`
    -   `feat-{task-name}` 브랜치 자동 생성 및 체크아웃.
    -   `CLAUDE.md`와 `docs/` 내용을 **가드레일**로 주입.
    -   Step 완료 시 자동으로 `feat` 및 `chore` 커밋을 분리하여 기록.
    -   실패 시 최대 **3회 재시도**하며 이전 에러를 피드백으로 제공.
-   `python3 scripts/execute.py {task-name} --push`
    -   실행 완료 후 생성된 브랜치를 원격 저장소(`origin`)에 자동으로 푸시합니다.

### B. 클로드 전용 스킬 (Commands)
`.claude/commands/`에 정의된 마크다운 파일들은 클로드가 수행할 수 있는 "특수 능력"입니다.
-   `harness.md`: 클로드에게 "하네스 구조를 어떻게 설계하고 세팅하는가"에 대한 전문 가이드를 제공합니다.
-   `review.md`: 클로드에게 "작성된 코드가 프로젝트 규칙을 준수하는지 리뷰하는 방법"을 지시합니다.

### C. 프로젝트 기본 명령어
`CLAUDE.md`에 기술된 표준 명령어들입니다.
-   `npm run dev`: 개발 서버 실행.
-   `npm run build`: 프로덕션 빌드 (Step 완료 시 검증용으로 사용).
-   `npm run test`: 테스트 코드 실행 (Acceptance Criteria의 핵심 지표).
