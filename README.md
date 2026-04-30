# Claude Code Configuration

Personal configuration files for [Claude Code](https://claude.com/claude-code) - Anthropic's official CLI for Claude.

## 📁 Repository Structure

This repository contains reusable configurations that enhance Claude Code's capabilities:

### 🤖 Agents (`agents/`)
Specialized AI agents for specific workflows:
- **analysis-planner** - Analysis design planning (目的・仮説・データ要件)
- **architect** - Code design, implementation planning and architectural decisions
- **tdd-guide** - Test-driven development guidance
- **code-reviewer** - Language-agnostic base code review (parallel with python-reviewer & sql-reviewer)
- **python-reviewer** - Python-specific review
- **sql-reviewer** - BigQuery SQL review (cost, security, correctness)
- **security-reviewer** - Security analysis
- **refactor-cleaner** - Dead code cleanup
- **doc-updater** - Documentation maintenance and Notion writing (analysis plans & reports)
- **analysis-reporter** - Conclusion structuring for PPDAC cycle (analysis results aggregation)
- **adversary** - PPDAC quality gate and red team analysis (critical review of Problem/Plan/Conclusion phases)

### 📋 Rules (`rules/`)
Context-specific guidelines organized by domain:
- `common/` - Cross-cutting concerns (git workflow, agents, security)
- `python/` - Python development standards
- `sql/` - SQL and database guidelines

### 🛠️ Skills (`skills/`)
Reusable skill definitions for specific tasks:
- `ppdac-workflow/` - PPDAC data analysis workflow (start from any phase, auto-detects state) with agent orchestration and quality gates
- `python-patterns/` - Python idioms and patterns
- `python-testing/` - Testing strategies
- `search-first/` - Code search workflows
- `security-review/` - Security analysis procedures
- `security-scan/` - Automated security scanning
- `strategic-compact/` - Code optimization
- `tdd-workflow/` - TDD process guidance