# Agent Orchestration

## Available Agents

Located in `~/.claude/agents/`:

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| `planner` | Implementation planning | Complex features, refactoring |
| `architect` | System design | Architectural decisions |
| `tdd-guide` | Test-driven development | New features, bug fixes |
| `code-reviewer` | Code review | After writing code |
| `python-reviewer` | Python-specific review | `.py` / `.ipynb` changes |
| `database-reviewer` | BigQuery review | SQL / schema changes |
| `security-reviewer` | Security analysis | Before commits with secrets/auth |
| `refactor-cleaner` | Dead code cleanup | Code maintenance |
| `doc-updater` | Documentation | Updating docs/reports |

## Proactive Agent Usage

No user prompt needed — activate automatically:
1. Complex feature requests → **planner**
2. Code just written/modified → **code-reviewer**
3. Bug fix or new feature → **tdd-guide**
4. Architectural decision → **architect**

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
