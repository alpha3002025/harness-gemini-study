

```plain
project/
├── CLAUDE.md                    ← 프로젝트 헌법
├── docs/
│   ├── PRD.md                   ← 뭘 만드는지
│   ├── ARCHITECTURE.md          ← 어떻게 만드는지
│   ├── ADR.md                   ← 왜 이렇게 만드는지
│   └── UI_GUIDE.md              ← 어떻게 보여야 하는지 (선택)
├── .claude/
│   ├── commands/
│   │   ├── harness.md           ← /harness — 원스톱 실행
│   │   └── review.md            ← /review — 규칙 기반 리뷰
│   └── settings.json            ← hooks 설정
├── scripts/
│   ├── execute.py               ← Phase 순차 실행 + 상태 관리
│   └── hooks/
│       ├── tdd-guard.sh         ← 테스트 없으면 구현 차단
│       ├── dangerous-cmd-guard.sh ← 위험 명령어 차단
│       └── circuit-breaker.sh   ← 반복 에러 감지
└── phases/                      ← Phase 파일 + 실행 상태
```

(1) docs/ - 프로젝트의 뇌
- 마크다운 4개로 프로젝트 전체를 설명
- PRD.md, ARCHITECTURE.md, ADR.md, UI_GUIDE.md

(2) CLAUDE.md - 프로젝트의 헌법
- CRITICAL 규칙 + TDD

(3) /harness + execute.py - 실행엔진
- Phase 분리 + 순차 실행 + 상태 관리

(4) Hooks - 자동 검증
- TDD 가드 + 위험 명령어 차단 + 서킷 브레이커


---








