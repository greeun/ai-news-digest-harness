# AI 뉴스 다이제스트 하네스

최신 AI 뉴스를 웹에서 수집하고, 읽기 쉬운 한국어로 요약하여, 정갈한 디자인의 단일 HTML 페이지로 제공하는 Claude Code 스킬입니다. Anthropic의 [Harness Design for Long-Running Application Development](https://www.anthropic.com/engineering/harness-design-long-running-apps) (Prithvi Rajasekaran, 2026)에서 제시한 **Planner → Generator → Evaluator** 하네스 패턴을 적용합니다.

## 작동 방식

```
사용자: "AI 뉴스 정리해줘"
        │
        ▼
┌─────────────┐     spec.md      ┌─────────────┐
│   Planner   │ ──────────────▶  │  Generator   │
│  (웹 검색)  │                  │ (HTML 제작)  │
└─────────────┘                  └──────┬───────┘
        │                               │
   사용자가 spec                generator_report.md
   확인 후 진행                         │
                                        ▼
                                ┌─────────────┐
                                │  Evaluator   │
                                │ (품질 검증)  │
                                └──────┬───────┘
                                       │
                              critique.md (PASS/FAIL)
                                       │
                        ┌──────────────┴──────────────┐
                        │                             │
                      PASS                          FAIL
                        │                             │
                        ▼                             ▼
               ai-news-digest-              새 Generator에
               {datetime}.html              critique.md 전달
                                            (최대 3회 반복)
```

### 세 에이전트의 구조적 분리

| 에이전트 | 역할 | 입력 | 출력 |
|---------|------|------|------|
| **Planner** | WebSearch/WebFetch로 실제 AI 뉴스 리서치 | 사용자 요청 + 오늘 날짜 | `spec.md` |
| **Generator** | Self-contained HTML 페이지 제작 | `spec.md` (+ 재시도 시 `critique.md`) | `ai-news-digest-{datetime}.html` |
| **Evaluator** | 루브릭 기반 증거 채점 | `spec.md` + HTML + `generator_report.md` | `critique.md` |

에이전트는 서로의 추론 과정을 볼 수 없습니다. **오직 파일을 통해서만** 소통합니다.

## 적용된 하네스 원칙

원문 아티클의 6가지 핵심 원칙을 인코딩합니다:

1. **구조적 역할 분리** — 생성과 평가를 별도 서브에이전트 프로세스로 실행 (GAN 구조에서 영감)
2. **파일 기반 핸드오프** — `spec.md`, `sprint_contract.md`, `generator_report.md`, `critique.md`
3. **구현 전 스프린트 계약** — Generator가 코딩 전에 범위와 검증 항목을 합의
4. **압축 대신 컨텍스트 리셋** — 압축은 연속성은 유지하지만 불안은 해소 못 함; 리셋은 둘 다 제거
5. **증거 기반 루브릭 평가** — "괜찮아 보인다"는 절대 통과가 아님
6. **모든 컴포넌트는 가정을 인코딩** — 모델 업그레이드 시 한 번에 하나씩 제거; 급진적 단순화는 실패

### V2 진화 교훈 반영

- **Simplified Harness** (스프린트 분할 없음) — Opus 4.6 기준 단일 아티팩트에 적합
- **Generator 전략적 판단** — 점수 상승 추세면 개선(refine), 정체면 피봇(pivot)
- **비선형 품질 향상** — 중간 이터레이션이 마지막보다 나을 수 있음; 모든 버전을 `-v{N}.html`로 보존
- **Evaluator 경계 개념** — 모델 능력 범위 내 작업에는 Evaluator 생략 가능
- **스타일 자석 방지** — 디자인 기준에 참조어 대신 품질 기술어 사용

## 평가 루브릭

| 기준 | 가중치 | 근거 |
|------|--------|------|
| 콘텐츠 완성도 | 1x | Claude가 잘 처리하는 영역 |
| **요약 품질** | **2x** | 한국어 품질이 가장 편차가 큰 영역 |
| **시각 디자인 품질** | **2x** | Claude가 기본적으로 제네릭 HTML을 생성하는 영역 |
| 기술적 정확성 | 1x | Claude가 안정적으로 유효한 HTML 생성 |
| 인터랙티비티 | 1x | 구체적 기능 검증 항목 |
| 접근성 & 완성도 | 1x | 중요하지만 주요 차별화 요소는 아님 |

**통과 기준**: 모든 항목 4/5 이상, 2x 가중 항목(요약 품질, 디자인 품질) 둘 다 4/5 이상.

## 산출물

단일 self-contained HTML 파일 (`ai-news-digest-{YYYYMMDD-HHmmss}.html`):

- 브라우저에서 직접 열기 가능 (`file://` 프로토콜)
- 7-12개 AI 뉴스 항목을 한국어로 요약
- 다크 모드 토글, 카테고리 필터, 반응형 레이아웃
- 임베디드 CSS/JS만 사용 (Google Fonts 외 외부 의존성 없음)

## 설치

`ai-news-digest-harness/` 폴더를 Claude Code 스킬 디렉토리에 복사합니다:

```
~/.claude/skills/ai-news-digest-harness/
```

## 사용법

다음 문구로 스킬을 트리거합니다:

```
AI 뉴스 정리해줘
오늘의 AI 소식
최신 AI 뉴스 모아줘
AI news digest
daily AI news
```

## 파일 구조

```
ai-news-digest-harness/
├── SKILL.md                        # 오케스트레이션 로직 + 원칙
├── README.md                       # 영문 문서
├── README.ko.md                    # 한국어 문서 (이 파일)
└── references/
    ├── planner-prompt.md           # 뉴스 리서치 + spec 작성
    ├── generator-prompt.md         # HTML 페이지 생성
    ├── evaluator-prompt.md         # 루브릭 기반 QA + 증거 수집
    └── rubric.md                   # 6개 평가 기준, 점수 가이드, 캘리브레이션
```

## 요구 사항

- [Claude Code](https://claude.ai/claude-code) CLI
- Claude Opus 4.6 (권장) 또는 Sonnet 4.5+
- 인터넷 접속 (뉴스 수집을 위한 WebSearch/WebFetch)

## 크레딧

Anthropic의 [Harness Design for Long-Running Application Development](https://www.anthropic.com/engineering/harness-design-long-running-apps) (Prithvi Rajasekaran, 2026년 3월) 기반. `skill-wizard-harness` 템플릿으로 생성.

## 라이선스

MIT
