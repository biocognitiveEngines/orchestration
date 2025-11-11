# Claude Code PR Security Review Setup (Optional)

This GitHub Action automatically comments on PRs with security analysis from Claude.

## Setup Instructions

### 1. Add GitHub Action Workflow

Create `.github/workflows/claude-pr-review.yml`:

```yaml
name: Claude PR Security Review

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  security-review:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      contents: read

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Claude PR Review
        uses: anthropics/claude-code-pr-review@v1
        with:
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### 2. Configure Anthropic API Key

1. Get an API key from: https://console.anthropic.com/settings/keys
2. Add to GitHub repository secrets:
   - Go to: Repository → Settings → Secrets and variables → Actions
   - Click "New repository secret"
   - Name: `ANTHROPIC_API_KEY`
   - Value: Your API key
   - Click "Add secret"

### 3. What It Does

The action will:
- ✅ Analyze code changes in PRs for security issues
- ✅ Check for common vulnerabilities (XSS, SQL injection, etc.)
- ✅ Review authentication and authorization logic
- ✅ Identify potential data leaks or unsafe operations
- ✅ Post findings as PR comments

### 4. Example Review Output

```markdown
## 🔒 Security Review by Claude

### ⚠️ Findings

**High Priority:**
- `src/auth.js:45` - Potential SQL injection vulnerability in user query
- `src/api.js:120` - Missing authentication check on sensitive endpoint

**Medium Priority:**
- `src/utils.js:78` - Using eval() with user input (security risk)

**Low Priority:**
- `src/config.js:12` - API key stored in plain text (consider env variable)

### ✅ Good Practices Observed
- Input validation on form submissions
- HTTPS enforcement
- CSRF protection enabled
```

### 5. Customization Options

You can customize the review focus by adding parameters:

```yaml
- name: Claude PR Review
  uses: anthropics/claude-code-pr-review@v1
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    focus-areas: 'security,performance,best-practices'
    severity-threshold: 'medium'  # Only report medium+ severity
    exclude-paths: 'tests/**,docs/**'  # Skip certain paths
```

## Benefits

- **Automated Security Review**: Every PR gets reviewed for security issues
- **Educational**: Learn from security feedback on your code
- **Complementary**: Works alongside your existing CI/tests
- **Non-Blocking**: Comments don't block merging (use as advisory)

## Cost Considerations

- Uses Anthropic API credits per PR review
- Typical cost: ~$0.01-0.10 per PR depending on size
- Can limit with `max-tokens` parameter if needed

## Disabling for Specific PRs

Add `[skip claude-review]` to PR title to skip the review.

## References

- Official Action: https://github.com/marketplace/actions/claude-code-pr-review
- Anthropic Blog: https://www.anthropic.com/news/claude-code-pr-review
- API Docs: https://docs.anthropic.com/claude/docs
