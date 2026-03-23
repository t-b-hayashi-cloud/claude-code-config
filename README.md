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

### 📋 Rules (`rules/`)
Context-specific guidelines organized by domain:
- `common/` - Cross-cutting concerns (git workflow, agents, security)
- `python/` - Python development standards
- `sql/` - SQL and database guidelines

### 🛠️ Skills (`skills/`)
Reusable skill definitions for specific tasks:
- `database-migrations/` - Database schema management
- `ppdac-workflow/` - PPDAC data analysis workflow with agent orchestration
- `postgres-patterns/` - PostgreSQL best practices
- `python-patterns/` - Python idioms and patterns
- `python-testing/` - Testing strategies
- `search-first/` - Code search workflows
- `security-review/` - Security analysis procedures
- `security-scan/` - Automated security scanning
- `strategic-compact/` - Code optimization
- `tdd-workflow/` - TDD process guidance
- `verification-loop/` - Quality assurance cycles

## 🚀 Usage

### Installation

1. Clone this repository:
   ```bash
   git clone <your-repo-url> ~/.claude-public
   ```

2. Copy desired configurations to your Claude Code directory:
   ```bash
   # Copy all agents
   cp -r ~/.claude-public/agents/* ~/.claude/agents/

   # Copy all rules
   cp -r ~/.claude-public/rules/* ~/.claude/rules/

   # Copy all skills
   cp -r ~/.claude-public/skills/* ~/.claude/skills/
   ```

3. Or symlink for automatic updates:
   ```bash
   ln -s ~/.claude-public/agents ~/.claude/agents
   ln -s ~/.claude-public/rules ~/.claude/rules
   ln -s ~/.claude-public/skills ~/.claude/skills
   ```

### Customization

Feel free to fork and modify these configurations for your needs:
- Agents: Adjust personas, instructions, and workflows
- Rules: Add project-specific guidelines
- Skills: Create new skill definitions

## 📖 Documentation

Each component includes inline documentation:
- Agents: See individual `.md` files in `agents/`
- Rules: Context files in `rules/*/`
- Skills: `SKILL.md` in each skill directory

## 🔒 Security Note

This repository contains **only public configuration files**. Sensitive data (credentials, history, projects) is never included. See `.gitignore` for exclusion rules.

## 📄 License

MIT License (or your preferred license)

## 🤝 Contributing

Contributions welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Submit a pull request with clear descriptions

## ⚠️ Disclaimer

This is a personal configuration repository. Configurations may reflect opinionated workflows and may not suit all use cases. Use at your own discretion.

---

**Note**: This repository is not officially affiliated with Anthropic. Claude Code is a product of Anthropic.
