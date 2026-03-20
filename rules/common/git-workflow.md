# Git & Development Workflow

## Feature Implementation Workflow

0. **Research & Reuse** _(mandatory before any new implementation)_
   - Search PyPI/conda for existing packages before writing utility code
   - Use `gh search code` to find existing implementations
   - Prefer battle-tested libraries over hand-rolled solutions

1. **Plan First**
   - Use **planner** agent for complex features
   - Identify dependencies and risks, break down into phases

2. **TDD Approach**
   - Use **tdd-guide** agent
   - Write tests first (RED) → implement (GREEN) → refactor (IMPROVE)
   - Verify 80%+ coverage

3. **Code Review**
   - Use **code-reviewer** agent after writing code
   - Address CRITICAL and HIGH issues

4. **Commit & Push**
   - Follow conventional commits format (see below)

## Commit Message Format

```
<type>: <description>

<optional body>
```

Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`

Note: Attribution disabled globally via `~/.claude/settings.json`.

## Pull Request Workflow

When creating PRs:
1. Analyze full commit history (`git diff [base-branch]...HEAD`)
2. Draft comprehensive PR summary
3. Include test plan with TODOs
4. Push with `-u` flag if new branch
