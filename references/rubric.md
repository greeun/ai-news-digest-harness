# Evaluation Rubric — AI News Digest

## Criteria

| # | Criterion | What it measures | Weight | Why this weight |
|---|-----------|-----------------|--------|-----------------|
| 1 | **Content Completeness** | All news items from spec present, correct data, no placeholders | 1x | Binary check — Claude is good at completeness |
| 2 | **Summary Quality** | Korean clarity, readability, natural phrasing, proper use of technical terms with English originals, informativeness | 2x | Claude tends toward stilted Korean and mechanical summaries — this is where quality varies most |
| 3 | **Visual Design Quality** | Modern, clean, magazine-style layout; clear typography hierarchy; proper color usage; adequate whitespace; professional feel | 2x | Claude defaults to generic, unstyled HTML — design quality needs extra pressure |
| 4 | **Technical Correctness** | Valid HTML5, working embedded CSS/JS, responsive viewport, self-contained (works on file://), no broken dependencies | 1x | Claude reliably produces valid HTML |
| 5 | **Interactivity** | Dark mode toggle works, category filter functions, hover effects present, smooth CSS transitions | 1x | Concrete functional checks — pass/fail |
| 6 | **Accessibility & Polish** | Semantic HTML (article, section, heading hierarchy), link targets (_blank + noopener), aria attributes, footer, meta tags | 1x | Important but not the primary differentiator |

## Scoring Guide

### Score 5 — Exceptional
- Beyond spec requirements
- Would impress a senior front-end developer or Korean tech editor
- Zero issues found

### Score 4 — Good
- Meets all spec requirements
- Minor polish opportunities but nothing blocking
- Professional quality

### Score 3 — Adequate
- Meets most requirements but has noticeable gaps
- Functional but not polished
- Needs one more pass

### Score 2 — Below Standard
- Missing significant spec items
- Noticeable bugs or visual issues
- Needs substantial rework

### Score 1 — Unacceptable
- Placeholder content, broken rendering, missing features
- Requires complete redo

## Passing Threshold

- **All criteria must score >= 4**
- **2x-weighted criteria (Summary Quality, Visual Design Quality) must both score >= 4**
- If any criterion scores < 4: FAIL

## Calibration Notes

- **Summary Quality**: Watch for these Korean-specific issues:
  - Unnatural sentence endings (e.g., "~하는 것이다" repeated)
  - Over-literal translations from English sources
  - Missing context that makes the summary unclear to a general reader
  - Technical terms without English originals on first use

- **Visual Design Quality**: Watch for these common AI-HTML pitfalls:
  - No visual hierarchy (everything same size/weight)
  - Default browser styling leaking through
  - Colors that clash or lack contrast
  - No hover states or transitions (feels static/dead)
  - Cards that are just bordered boxes with no depth
  - Mobile layout that's just "everything stacked" with no thought

- **Late-stage creative leaps**: If the Generator produces a genuinely innovative design approach on a later iteration (e.g., timeline layout, interactive data viz), evaluate it on merit against the rubric — don't penalize for departing from a conventional card layout if the result scores higher on design quality.
