# Security Guidelines

## Secret Management

- NEVER hardcode secrets in source code
- ALWAYS use environment variables or `.env` (gitignored)
- Validate required secrets are present at startup
- Rotate any secrets that may have been exposed

Python example:
```python
import os
from dotenv import load_dotenv

load_dotenv()
api_key = os.environ["API_KEY"]  # Raises KeyError if missing
```

## Commit Pre-checks

Before ANY commit:
- [ ] No hardcoded secrets (API keys, passwords, tokens)
- [ ] `.env` is gitignored (never `.env.example`)
- [ ] SQL queries use parameterized queries (no f-string interpolation)
- [ ] Error messages don't leak sensitive data

## Security Scanning

```bash
bandit -r src/    # Python static security analysis
```

## Security Response Protocol

If security issue found:
1. STOP immediately
2. Use **security-reviewer** agent
3. Fix CRITICAL issues before continuing
4. Rotate any exposed secrets
