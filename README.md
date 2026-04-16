# AI News Digest Harness

A Claude Code skill that collects trending AI news from the web, summarizes them in clear Korean, and delivers a polished, self-contained HTML page — powered by the **Planner → Generator → Evaluator** harness pattern from Anthropic's [Harness Design for Long-Running Application Development](https://www.anthropic.com/engineering/harness-design-long-running-apps) (Prithvi Rajasekaran, 2026).

## How It Works

```
User: "AI 뉴스 정리해줘"
        │
        ▼
┌─────────────┐     spec.md      ┌─────────────┐
│   Planner   │ ──────────────▶  │  Generator   │
│ (WebSearch) │                  │ (HTML build) │
└─────────────┘                  └──────┬───────┘
        │                               │
   User confirms                generator_report.md
   spec before                         │
   proceeding                          ▼
                                ┌─────────────┐
                                │  Evaluator   │
                                │ (QA + grade) │
                                └──────┬───────┘
                                       │
                              critique.md (PASS/FAIL)
                                       │
                        ┌──────────────┴──────────────┐
                        │                             │
                      PASS                          FAIL
                        │                             │
                        ▼                             ▼
               ai-news-digest-              Fresh Generator
               {datetime}.html              with critique.md
                                            (up to 3 rounds)
```

### Three Agents, Structurally Separated

| Agent | Role | Input | Output |
|-------|------|-------|--------|
| **Planner** | Research real AI news via WebSearch/WebFetch | User prompt + today's date | `spec.md` |
| **Generator** | Build a self-contained HTML page | `spec.md` (+ `critique.md` on retry) | `ai-news-digest-{datetime}.html` |
| **Evaluator** | Grade against rubric with evidence | `spec.md` + HTML + `generator_report.md` | `critique.md` |

Agents never see each other's reasoning. They communicate **exclusively through files**.

## Harness Principles Applied

This skill encodes six principles from the original article:

1. **Structural role separation** — Generation and evaluation are different subagent processes (GAN-inspired)
2. **File-based handoffs** — `spec.md`, `sprint_contract.md`, `generator_report.md`, `critique.md`
3. **Sprint contracts before implementation** — Generator proposes scope + checks before coding
4. **Context resets over compaction** — Compaction preserves continuity but not anxiety; resets eliminate both
5. **Rubric-based evaluation with evidence** — "It looks fine" is never a pass
6. **Every component encodes an assumption** — Remove one at a time on model upgrade; radical simplification fails

### V2 Evolution Lessons

- **Simplified Harness** (no per-sprint decomposition) — appropriate for single-artifact output with Opus 4.6
- **Generator strategic decisions** — refine current direction if scores trend up; pivot if stagnant
- **Non-linear quality** — middle iterations can outperform the last; all versions preserved as `-v{N}.html`
- **Evaluator boundary concept** — evaluator is optional when task is well within model capability
- **Style magnet prevention** — design criteria use quality descriptors, not reference words

## Evaluation Rubric

| Criterion | Weight | Rationale |
|-----------|--------|-----------|
| Content Completeness | 1x | Claude handles completeness well |
| **Summary Quality** | **2x** | Korean NLP quality varies most here |
| **Visual Design Quality** | **2x** | Claude defaults to generic HTML |
| Technical Correctness | 1x | Claude reliably produces valid HTML |
| Interactivity | 1x | Concrete functional checks |
| Accessibility & Polish | 1x | Important but not the differentiator |

**Pass threshold**: All criteria >= 4/5, both 2x-weighted criteria >= 4/5.

## Output

A single, self-contained HTML file (`ai-news-digest-{YYYYMMDD-HHmmss}.html`) that:

- Works when opened directly in a browser (`file://` protocol)
- Contains 7-12 AI news items summarized in Korean
- Features dark mode toggle, category filters, responsive layout
- Uses embedded CSS/JS only (no external dependencies except Google Fonts)

## Installation

Copy the `ai-news-digest-harness/` folder into your Claude Code skills directory:

```
~/.claude/skills/ai-news-digest-harness/
```

## Usage

Trigger the skill with any of these phrases:

```
AI 뉴스 정리해줘
오늘의 AI 소식
최신 AI 뉴스 모아줘
AI news digest
daily AI news
```

## File Structure

```
ai-news-digest-harness/
├── SKILL.md                        # Orchestration logic + principles
├── README.md                       # This file
├── README.ko.md                    # Korean documentation
└── references/
    ├── planner-prompt.md           # News research + spec writing
    ├── generator-prompt.md         # HTML page generation
    ├── evaluator-prompt.md         # Rubric-based QA with evidence
    └── rubric.md                   # 6 criteria, scoring guide, calibration
```

## Requirements

- [Claude Code](https://claude.ai/claude-code) CLI
- Claude Opus 4.6 (recommended) or Sonnet 4.5+
- Internet access for WebSearch/WebFetch (news collection)

## Credits

Based on Anthropic's [Harness Design for Long-Running Application Development](https://www.anthropic.com/engineering/harness-design-long-running-apps) by Prithvi Rajasekaran (March 2026). Skill created using the `skill-wizard-harness` template.

## License

MIT
