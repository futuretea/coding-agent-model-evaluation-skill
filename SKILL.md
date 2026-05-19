# Coding Agent Model Evaluation Skill

## Skill purpose

Use this skill to evaluate and rank newly released large language models for **long-duration autonomous coding-agent work**.

The default target scenario is:

> The model independently enters a repository, reads code, plans changes, edits files, runs commands, inspects logs, fixes failing tests, iterates, and delivers a working result.

This skill is not primarily for ranking models by pure code generation. It evaluates whether a model can complete an engineering workflow end-to-end.

---

## When to use this skill

Use this skill when the user asks to:

- Compare models for coding-agent usage.
- Evaluate a newly released model using benchmark data.
- Rank models for long-context, multi-step software engineering tasks.
- Decide whether a model should be used as a primary autonomous coding agent, a code-generation assistant, a reviewer, or a debugging helper.
- Apply a “wooden bucket” / bottleneck theory to model evaluation.
- Build or update a model leaderboard for coding, planning, terminal usage, SWE tasks, or long-running agent workflows.

Do not use this skill for general-purpose chatbot ranking unless the user explicitly wants coding-agent suitability.

---

## Core evaluation principle

For long-duration coding-agent tasks, do **not** rank models by their strongest benchmark alone.

Use this principle:

> Short tasks are often determined by a model’s strongest capability. Long autonomous engineering tasks are determined by the weakest capability on the critical path.

The critical path is usually:

```text
requirements understanding
→ planning and design
→ repository comprehension
→ code editing
→ command execution
→ log/error interpretation
→ debugging and repair
→ retesting
→ final delivery
```

A model that writes excellent code but cannot reliably run commands, interpret failures, or maintain state across a long task should not be ranked as a top autonomous coding agent.

---

## Required inputs

Collect as many of the following as available:

1. Model name and version.
2. Benchmark scores.
3. Context window size and long-context quality scores.
4. Tool/function/terminal support.
5. Pricing and speed information if the user cares about production use.
6. Any private evaluation results from the user’s own repositories.
7. Evidence source and date for each score when available.

If benchmark data may be stale or the model was released recently, retrieve current data from reliable sources before scoring. Prefer official model reports, benchmark leaderboards, public eval repos, and reproducible third-party evaluations over social media claims.

---

## Evaluation dimensions

Score each model from 0 to 100 on seven dimensions.

| Dimension | Default weight | Meaning |
|---|---:|---|
| A. Code implementation | 20% | Can the model write correct, maintainable code? |
| B. Engineering repair | 20% | Can it fix real repository issues and produce working patches? |
| C. Tool and terminal execution | 20% | Can it use tools, shell commands, test runners, logs, and multi-step workflows? |
| D. Long-context and state retention | 15% | Can it remember repository structure, constraints, and previous changes over long tasks? |
| E. Planning and design | 10% | Can it decompose tasks, choose architecture, and avoid chaotic edits? |
| F. Debugging and self-correction | 10% | Can it diagnose failed tests, revise hypotheses, and avoid repeated blind fixes? |
| G. Stability, cost, speed, and availability | 5% | Is it practical for long-running agent use? |

Default scenario weights target autonomous coding-agent work. Adjust the weights only if the user’s scenario differs.

---

## Benchmark mapping

Use these benchmark families to estimate each dimension.

### A. Code implementation

Useful signals:

- LiveCodeBench
- Codeforces / competitive programming rating
- HumanEval / MBPP, as weak supplementary signals
- Aider Polyglot or other multi-language coding benchmarks
- Language-specific coding evals relevant to the user’s stack

Interpretation:

- High LiveCodeBench suggests strong general code generation.
- High Codeforces suggests strong algorithmic reasoning, implementation detail, and edge-case handling.
- High HumanEval alone is insufficient because many modern models saturate it.

### B. Engineering repair

Useful signals:

- SWE-bench Verified
- SWE Pro
- SWE Multilingual
- RepoBench or repository-level coding tasks
- Private bug-fix tasks with failing tests

Interpretation:

- SWE-style benchmarks are more important than algorithm benchmarks for repository work.
- A model with high coding scores but weak SWE scores may be a strong code generator but a weak engineering agent.

### C. Tool and terminal execution

Useful signals:

- Terminal Bench
- Toolathlon
- HLE with tools
- MCPAtlas or MCP tool-use benchmarks
- Browser/search-agent benchmarks
- Private tests involving shell commands, package installation, build failures, and CI logs

Interpretation:

- This dimension is critical for the default scenario.
- A model with weak tool/terminal performance should not be ranked as a top autonomous coding agent, even if it writes strong code.

### D. Long-context and state retention

Useful signals:

- MRCR 1M
- CorpusQA 1M
- Needle-in-a-Haystack
- LongBench
- Repository-level long-context tasks
- Private long-running tasks across many files and iterations

Interpretation:

- Context window size is not the same as long-context competence.
- Prefer quality metrics showing that the model can retrieve and use the right information from long context.

### E. Planning and design

Useful signals:

- GPQA
- MMLU-Pro
- HLE
- BrowseComp
- Apex / planning benchmarks if available
- Private system-design and task-decomposition tests

Interpretation:

- Strong reasoning and search/comparison skills help the model plan work before editing files.
- A model may be good at architecture discussion while still being weaker at autonomous implementation.

### F. Debugging and self-correction

Useful signals:

- SWE Pro
- SWE-bench Verified
- Terminal Bench
- HLE with tools
- Private failing-test repair tasks

Interpretation:

- Observe behavior after failure, not only final pass/fail.
- Strong models form hypotheses, inspect evidence, make targeted changes, and retest.
- Weak models repeatedly guess, modify unrelated code, or claim success without verification.

### G. Stability, cost, speed, and availability

Useful signals:

- Price per input/output token
- Long-context pricing
- Latency
- Rate limits
- API stability
- Tool support
- IDE/CLI/MCP integration
- Privacy and deployment constraints

Interpretation:

- This dimension has low default weight, but it can become a hard constraint in production use.

---

## Score normalization

Before combining benchmark results, normalize them to a 0–100 scale.

### Percentage metrics

For metrics already reported as percentages or pass rates:

```text
normalized_score = raw_percentage
```

Example:

```text
SWE Verified 80.6 → 80.6
```

### Rating or Elo-style metrics

For rating metrics such as Codeforces, do not average the raw rating with percentage scores.

Use either cohort normalization:

```text
normalized_score = 100 × (model_rating - cohort_min_rating) / (cohort_max_rating - cohort_min_rating)
```

Or use a fixed practical scale:

```text
Codeforces 2400 → 60
Codeforces 2800 → 80
Codeforces 3200 → 100
```

Cap scores above 100 unless the user explicitly wants an open-ended scale.

### Missing data

Do not treat missing data as zero by default. Also do not let missing data silently benefit a model.

Use one of these approaches:

1. Estimate from closely related benchmarks and mark the score as low-confidence.
2. Leave the dimension unscored and apply a confidence penalty.
3. If the missing dimension is critical, downgrade the model’s final recommendation.

Always report material missing data.

---

## Dimension scoring formulas

Use these as default formulas when the relevant data exists.

### A. Code implementation

```text
A = 0.45 × LiveCodeBench
  + 0.35 × Codeforces_normalized
  + 0.20 × multi_language_coding_score
```

If multi-language data is missing, redistribute its weight across the available coding benchmarks and reduce confidence.

### B. Engineering repair

```text
B = 0.40 × SWE_Verified
  + 0.35 × SWE_Pro
  + 0.25 × SWE_Multilingual
```

If only SWE Verified is available, use it as the main signal but mark B as incomplete.

### C. Tool and terminal execution

```text
C = 0.45 × Terminal_Bench
  + 0.30 × Toolathlon
  + 0.15 × HLE_with_tools
  + 0.10 × MCP_or_tool_ecosystem_score
```

If Terminal Bench is missing, the model should not be confidently ranked as a top autonomous coding agent unless strong private tool-use results exist.

### D. Long-context and state retention

```text
D = 0.50 × MRCR_1M
  + 0.35 × CorpusQA_1M
  + 0.15 × repository_long_context_or_LongBench_score
```

If the model has a large context window but lacks quality scores, assign only a provisional score.

### E. Planning and design

```text
E = 0.30 × GPQA_or_HLE
  + 0.25 × MMLU_Pro
  + 0.25 × BrowseComp
  + 0.20 × private_planning_score
```

When no private planning score exists, redistribute its weight and reduce confidence.

### F. Debugging and self-correction

```text
F = 0.35 × SWE_Pro
  + 0.25 × Terminal_Bench
  + 0.20 × HLE_with_tools
  + 0.20 × private_failing_test_repair_score
```

Private failing-test repair data is especially valuable here.

### G. Stability, cost, speed, and availability

Use a practical 0–100 judgment score based on the user’s context.

Suggested rubric:

| Score range | Meaning |
|---:|---|
| 85–100 | Fast, stable, affordable enough, strong tooling, few limits |
| 70–84 | Usable for long tasks with minor tradeoffs |
| 55–69 | Usable but constrained by cost, speed, limits, or integration |
| 0–54 | Impractical for long-running agent use |

---

## Default total score

For autonomous coding-agent evaluation, compute:

```text
weighted_total =
  0.20 × A
+ 0.20 × B
+ 0.20 × C
+ 0.15 × D
+ 0.10 × E
+ 0.10 × F
+ 0.05 × G
```

Then apply bottleneck penalties.

---

## Bottleneck penalty

For the default scenario, the critical dimensions are:

```text
B = Engineering repair
C = Tool and terminal execution
D = Long-context and state retention
F = Debugging and self-correction
```

Compute:

```text
critical_min = min(B, C, D, F)
bottleneck_factor = min(1, critical_min / 75)
final_score = weighted_total × bottleneck_factor
```

This penalizes models that have strong peak performance but weak critical-path reliability.

---

## Hard downgrade rules

Apply these after calculating the score.

### Rule 1: Key dimension below 60

If any of B, C, D, or F is below 60:

```text
maximum final grade = B
```

### Rule 2: Key dimension below 50

If any of B, C, D, or F is below 50:

```text
not recommended as a primary long-duration autonomous coding agent
```

It may still be recommended as a secondary assistant for code generation, planning, review, or short tasks.

### Rule 3: Missing critical data

If a critical dimension has no benchmark or private evidence:

```text
recommendation confidence = medium or low
```

If multiple critical dimensions are missing, do not give an S-grade recommendation.

### Rule 4: Unsafe failure behavior

If private testing shows the model frequently fabricates test results, claims unverified success, or makes broad unrelated modifications:

```text
downgrade at least one grade
```

For autonomous coding, failure behavior is part of capability.

---

## Grade scale

Assign grades using both final score and hard rules.

| Grade | Final score | Meaning |
|---|---:|---|
| S | 85+ and no critical short board | Primary long-duration autonomous coding agent |
| A | 78–84 | Strong agent, suitable for most engineering tasks |
| B | 68–77 | Usable with supervision or model pairing |
| C | 55–67 | Best for short tasks, local implementation, or support roles |
| D | <55 | Not recommended for long-duration coding-agent work |

S-grade requires:

```text
final_score >= 85
B >= 75
C >= 75
D >= 70
F >= 70
no serious missing critical data
no unsafe failure behavior
```

---

## Scenario-specific weight presets

Use these presets when the user’s task differs from the default scenario.

### Preset 1: Pure code generation / algorithms / function implementation

| Dimension | Weight |
|---|---:|
| A. Code implementation | 40% |
| B. Engineering repair | 15% |
| C. Tool and terminal execution | 10% |
| D. Long-context and state retention | 10% |
| E. Planning and design | 10% |
| F. Debugging and self-correction | 10% |
| G. Stability, cost, speed, availability | 5% |

### Preset 2: Autonomous coding agent / long engineering task

| Dimension | Weight |
|---|---:|
| A. Code implementation | 20% |
| B. Engineering repair | 20% |
| C. Tool and terminal execution | 20% |
| D. Long-context and state retention | 15% |
| E. Planning and design | 10% |
| F. Debugging and self-correction | 10% |
| G. Stability, cost, speed, availability | 5% |

This is the default preset.

### Preset 3: Architecture design / technical planning / code review

| Dimension | Weight |
|---|---:|
| A. Code implementation | 15% |
| B. Engineering repair | 10% |
| C. Tool and terminal execution | 10% |
| D. Long-context and state retention | 20% |
| E. Planning and design | 30% |
| F. Debugging and self-correction | 10% |
| G. Stability, cost, speed, availability | 5% |

### Preset 4: Low-cost frequent small tasks

| Dimension | Weight |
|---|---:|
| A. Code implementation | 30% |
| B. Engineering repair | 15% |
| C. Tool and terminal execution | 10% |
| D. Long-context and state retention | 10% |
| E. Planning and design | 5% |
| F. Debugging and self-correction | 5% |
| G. Stability, cost, speed, availability | 25% |

---

## Private evaluation protocol

Public benchmarks are not enough. When possible, run private repository tests that match the user’s real work.

Use at least six task types:

| Task type | What it measures |
|---|---|
| New feature implementation | Requirements to working code |
| Bug fix with failing tests | Root-cause analysis and patch quality |
| Cross-file refactor | Repository comprehension and consistency |
| Dependency/build/CI failure | Terminal and environment handling |
| Code review | Design judgment and bug detection |
| Multi-round long task | State retention and goal stability |

Record these outcome metrics:

| Metric | Direction |
|---|---|
| Task success | Higher is better |
| Tests passed | Higher is better |
| Human interventions | Lower is better |
| Unrelated changes | Lower is better |
| Time to completion | Lower is better, if quality is maintained |
| Token or dollar cost | Lower is better, if quality is maintained |
| Verification honesty | Must not fabricate success |
| Quality of final explanation | Higher is better |

A model that fails but accurately identifies the blocker may be more useful than a model that falsely claims success.

---

## Recommended final report format

When presenting an evaluation, use this structure:

```text
Model: <model name>
Scenario: <default or selected preset>
Overall grade: <S/A/B/C/D>
Final score: <number>
Confidence: <high/medium/low>

Best role:
- Primary autonomous coding agent / code generator / debugger / architect / reviewer / low-cost assistant

Dimension scores:
A. Code implementation: <score> — <brief reason>
B. Engineering repair: <score> — <brief reason>
C. Tool and terminal execution: <score> — <brief reason>
D. Long-context and state retention: <score> — <brief reason>
E. Planning and design: <score> — <brief reason>
F. Debugging and self-correction: <score> — <brief reason>
G. Stability, cost, speed, availability: <score> — <brief reason>

Weighted total: <score>
Critical minimum: <score>
Bottleneck factor: <score>
Final score after bottleneck penalty: <score>

Core strengths:
1. ...
2. ...
3. ...

Critical short boards:
1. ...
2. ...

Recommendation:
- Use as: ...
- Avoid as: ...
- Best paired with: ...
```

---

## Ranking method for multiple models

When ranking several models:

1. Normalize all comparable benchmark scores.
2. Score each dimension.
3. Calculate weighted total.
4. Apply bottleneck penalty.
5. Apply hard downgrade rules.
6. Rank by final score.
7. Break ties using scenario relevance:
   - For autonomous coding agents, prefer higher C, then B, then F, then D, then A.
   - For pure code generation, prefer higher A, then B, then F.
   - For architecture/review, prefer higher E, then D, then A.
8. Report confidence and missing data.

Do not hide material uncertainty. If a model ranks highly only because several critical dimensions are missing, mark the ranking as provisional.

---

## Common mistakes to avoid

Avoid these mistakes:

- Ranking by a single benchmark.
- Treating Codeforces rating as a percentage.
- Treating a large context window as proof of good long-context reasoning.
- Treating missing data as neutral.
- Ignoring tool and terminal benchmarks for autonomous coding-agent tasks.
- Ignoring failure behavior in private tests.
- Averaging all benchmarks without regard to the task’s critical path.
- Overvaluing models that write impressive code but cannot verify it.

---

## Practical rule of thumb

For autonomous coding-agent work, evaluate in this order:

```text
1. Tool and terminal execution
2. SWE / engineering repair
3. Debugging and self-correction
4. Long-context and state retention
5. Code implementation
6. Planning and design
7. Cost, speed, and operational constraints
```

Final principle:

> A strong autonomous coding agent is not merely the model that writes the best code. It is the model least likely to fail on the weakest link of the engineering workflow.
