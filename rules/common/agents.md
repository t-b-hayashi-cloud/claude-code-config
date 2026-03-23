# Agent Orchestration

## Available Agents

Located in `~/.claude/agents/`:

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| `analysis-planner` | Analysis design planning | New analysis requests, hypothesis & data design |
| `architect` | Code design & implementation planning | Technical design, complex features, refactoring |
| `tdd-guide` | Test-driven development | New features, bug fixes |
| `code-reviewer` | Language-agnostic base code review | After writing code (parallel with python-reviewer & sql-reviewer) |
| `python-reviewer` | Python-specific review | `.py` / `.ipynb` changes |
| `sql-reviewer` | BigQuery SQL review | SQL file creation / modification |
| `security-reviewer` | Security analysis | Before commits with secrets/auth |
| `refactor-cleaner` | Dead code cleanup | Code maintenance |
| `doc-updater` | Documentation & Notion writing (analysis plans/reports) | Updating docs/reports, writing to Notion |
| `analysis-reporter` | Conclusion structuring for PPDAC cycle | Analysis Conclusion phase, aggregating results |

## Proactive Agent Usage

No user prompt needed — activate automatically:
1. New analysis request (目的・仮説・データ要件の設計) → **analysis-planner**
2. Complex code design or implementation → **architect**
3. Code just written/modified → **code-reviewer**
4. Bug fix or new feature → **tdd-guide**
5. Analysis Conclusion phase (results aggregation & reporting) → **analysis-reporter**

## Parallel Task Execution

ALWAYS use parallel Task execution for independent operations:

```
# GOOD: Launch in parallel
Agent 1: Security analysis of query module
Agent 2: Performance review of data pipeline
Agent 3: Type checking of utilities

# BAD: Sequential when unnecessary
First agent 1, then agent 2, then agent 3
```

## Multi-Perspective Analysis

For complex problems, split into sub-agents:
- Factual reviewer
- Senior data scientist
- Security expert
- Consistency reviewer
