# Coding Agent Model Evaluation Skill

A Claude Code skill for evaluating and ranking large language models specifically for **long-duration autonomous coding-agent work** — where the model independently enters a repository, reads code, plans changes, edits files, runs commands, inspects logs, fixes failing tests, and delivers a working result.

## Installation

```bash
cp SKILL.md ~/.claude/skills/coding-agent-model-evaluation.SKILL.md
```

Or place it in your project's `.claude/skills/` directory.

## Usage

Invoke in Claude Code when you need to:

- Compare models for coding-agent usage
- Evaluate a newly released model using benchmark data
- Rank models for long-context, multi-step software engineering tasks
- Decide a model's best role (primary agent, code generator, reviewer, debugger)

## Core Principle

> Short tasks are often determined by a model's strongest capability. Long autonomous engineering tasks are determined by the weakest capability on the critical path.

The evaluation scores models across seven dimensions weighted for autonomous coding work, then applies a bottleneck penalty that penalizes models with strong peaks but weak critical-path reliability.

## Evaluation Dimensions

| Dimension | Weight |
|---|---|
| A. Code implementation | 20% |
| B. Engineering repair (SWE tasks) | 20% |
| C. Tool and terminal execution | 20% |
| D. Long-context and state retention | 15% |
| E. Planning and design | 10% |
| F. Debugging and self-correction | 10% |
| G. Stability, cost, speed, availability | 5% |

## Grade Scale

| Grade | Final Score | Meaning |
|---|---|---|
| S | 85+ | Primary long-duration autonomous coding agent |
| A | 78–84 | Strong agent, suitable for most engineering tasks |
| B | 68–77 | Usable with supervision or model pairing |
| C | 55–67 | Best for short tasks or support roles |
| D | <55 | Not recommended for long-duration coding-agent work |

## Scenario Presets

The skill includes four weight presets: autonomous coding agent (default), pure code generation, architecture design / code review, and low-cost frequent small tasks.

## License

MIT
