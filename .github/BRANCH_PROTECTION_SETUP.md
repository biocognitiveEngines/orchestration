# Branch Protection Setup Guide

## Configure Branch Protection on GitHub

To enable safe automation with CI checks, you need to configure branch protection rules for the `main` branch.

### Steps to Configure:

1. **Navigate to Repository Settings**
   - Go to: `https://github.com/biocognitiveEngines/orchestration/settings/branches`
   - Or: Repository → Settings → Branches

2. **Add Branch Protection Rule**
   - Click "Add branch protection rule"
   - Branch name pattern: `main`

3. **Required Settings for Automation**

   #### ✅ Require a pull request before merging
   - ☑️ Enable this option
   - Require approvals: 0 (Codex will handle approval via policy)
   - ☑️ Dismiss stale pull request approvals when new commits are pushed
   - ☐ Require review from Code Owners (optional)

   #### ✅ Require status checks to pass before merging
   - ☑️ Enable this option
   - ☑️ Require branches to be up to date before merging
   - Add required status checks:
     - `test-python` (if using Python)
     - `test-node` (if using Node.js)
     - `test-go` (if using Go)
     - `security-scan` (recommended)

   #### ✅ Additional recommended settings
   - ☑️ Require conversation resolution before merging
   - ☑️ Require linear history (optional - keeps clean git history)
   - ☐ Do not allow bypassing the above settings (recommended for production)

4. **Save Changes**
   - Click "Create" or "Save changes"

## Verification

After setting up, test the protection by:

```bash
# Try to push directly to main (should fail)
git checkout main
echo "test" > test.txt
git add test.txt
git commit -m "test: direct push"
git push origin main  # Should be rejected
```

You should see an error indicating that direct pushes to `main` are not allowed.

## For Codex Automation

With these settings:
- Claude Code can create feature branches and open PRs
- CI runs automatically on all PRs
- Codex can merge PRs only when CI passes
- Direct pushes to `main` are blocked
- All changes go through PR review workflow

## Troubleshooting

**Issue**: Status checks don't appear in the list
- **Solution**: Push a PR first to trigger the workflow, then the checks will appear

**Issue**: Codex can't merge PRs
- **Solution**: Ensure Codex uses `gh` CLI with proper authentication and doesn't bypass protections

**Issue**: Want to allow administrators to bypass
- **Solution**: Uncheck "Do not allow bypassing the above settings" but use cautiously
