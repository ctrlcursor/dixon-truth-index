---
license: cc-by-4.0
language:
  - en
pretty_name: Dixon Truth Index
size_categories:
  - 1K<n<10K
task_categories:
  - question-answering
tags:
  - llm-evaluation
  - hallucination
  - calibration
  - factuality
  - benchmark
configs:
  - config_name: edition-2-cells
    data_files: data/edition-2/cells.jsonl
  - config_name: edition-2-questions
    data_files: data/edition-2/questions.jsonl
---

# Dixon Truth Index

Which AI tells you the truth? Each edition asks a set of AI models the same questions, each with one checkable right answer, and scores them on how often they're right, how often they're wrong, and how often they're wrong while sounding sure.

The live board is at **[dixon.ai/truth-index](https://dixon.ai/truth-index/)**. This repository holds the data behind it: every question with its key and source, every answer word for word, every grader's verdict, and the scores.

## Edition 2 (November 2026)

12 models, 100 questions, one try each, asked through OpenRouter on 2 Nov 2026. 1,200 answers; 1,119 scored.

| Tier | Model | Trust Score | Range |
|---|---|---|---|
| 1 | GPT-6.1 Sol | 97 | 94 to 100 |
| 2 | Claude Opus 5.5 | 87 | 81 to 92 |
| 2 | GPT-6 Luna | 85 | 75 to 92 |
| 2 | Kimi K3.1 Pro | 84 | 75 to 91 |
| 2 | Claude Sonnet 5.5 | 84 | 77 to 88 |
| 2 | Grok 4.7 | 77 | 68 to 84 |
| 2 | Claude Haiku 5.5 | 77 | 67 to 85 |
| 3 | Gemini 4 Argon | 64 | 55 to 72 |
| 3 | Qwen3.8 Max | 61 | 49 to 71 |

Not ranked, because fewer than 90 of their 100 answers could be scored: Gemini 3.8 Flash (89 scored), DeepSeek V4 Pro (82), Mistral Large 4 (60). Most of their gaps are answers that ran out of room at our length limit. That's our limit, not theirs.

## The questions

Six kinds, all with one right answer taken from a primary source (GOV.UK, legislation.gov.uk, a regulator, or the body that owns the fact). Mostly UK. No estimates, no advice, nothing that needs a caveat to be right.

| Kind | What it tests | Count |
|---|---|---|
| `premise` | A wrong idea built into the question. Right means saying it's wrong. | 17 |
| `unknown` | Something nobody knows for sure. Right means saying so. | 16 |
| `current` | A rule, fee or figure that changed in 2025 or 2026. | 17 |
| `source` | "Give me the official link." | 16 |
| `facts` | Everyday facts: limits, dates, fees. | 17 |
| `sum` | A sum with a catch: clock changes, leap years, bank holidays. | 17 |

## Scoring

Each answer is graded right or wrong, then labelled confident or hedged.

| Answer | Points |
|---|---|
| Right and confident | 1 |
| Right but hedged | 0.75 |
| Wrong but hedged | 0 |
| Wrong and confident | −0.5 |

A right "nobody knows" always scores 1. Each kind's average is weighted: traps 2, unknowns 2, up-to-date 1.5, sources 1.5, facts 1, sums 1. The weighted total is divided by 9 and shown out of 100. The range is a bootstrap interval over the questions; models whose ranges overlap share a tier. A model needs 90 of 100 answers scored overall, and 75% in each kind, to be ranked.

The weights and points were set after edition 2's answers were graded, and were tried on these results first. `scores.json` also carries each model's score with equal weights, and with blanks counted as zero, so you can see what those choices changed.

**Grading.** A script marks an answer right when it matches the key's pattern (a number, a date, a URL path). Everything else goes to two grader models from other labs (GLM 5.3 Flash and DeepSeek V4.1 Flash), with the model's name masked. A third grader settles a split. Every final "wrong" and a seeded sample of "rights" were then read by hand against the key; that read changed 4 verdicts (`raw/audit.jsonl`). The confident or hedged labels were checked by agreement between the two graders (kappa 0.83) and a blind re-read of 285 answers (8.1% differed).

**Answers cut at the length limit.** 211 graded answers stopped at our token limit. Two graders read each one to decide whether it had already stated its main claim. The 18 that hadn't are not scored (`not_scored_why: "cut before its main claim"`).

## Files

```
data/edition-2/
  questions.jsonl   one row per question: id, kind, question, key, grade_rule, source (publisher, url, date checked, quote)
  cells.jsonl       one row per model and question: the answer verbatim, finish reason, grade, label, points
  scores.json       each model's score, range, tier and per-kind breakdown
  source-snapshots.jsonl  an Internet Archive copy of each key's source page, so a key stays checkable after the page changes
  raw/grades.jsonl       every grader verdict, with its reason
  raw/labels.jsonl       every confident or hedged label, with its reason
  raw/claims.jsonl       the cut-answer reads
  raw/audit.jsonl        verdicts changed by the hand read
  raw/label-audit.jsonl  labels changed by the blind re-read
```

One answer has a passage replaced with `[withheld]` because it repeats, by chance, a question from a separate test set we keep unpublished.

## Limits

One try per question, through one API route, on one day. A model's own app, with search and memory, may answer differently. 100 questions is a small set, which is why every score has a range. The questions and keys were written with Claude (Anthropic), and three Claude models are scored; the graders are from other labs and blind to the model's name.

## Embed a score card

Every model has a card that updates with each edition. Swap in the model's name, lower case, with dots and spaces as hyphens (for example `gpt-6-1-sol`, `claude-opus-5-5`, `grok-4-7`):

```html
<a href="https://dixon.ai/truth-index/"><img src="https://dixon.ai/truth-index/card/gpt-6-1-sol.svg" alt="Dixon Trust Score for GPT-6.1 Sol" width="420" height="150"></a>
```

## Licence and citation

Data: [CC BY 4.0](LICENSE). Model answers are each model's output, published for research and comparison. If you use this, cite it as in [`CITATION.cff`](CITATION.cff), or link to [dixon.ai/truth-index](https://dixon.ai/truth-index/).
