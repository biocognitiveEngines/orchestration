0) What you’ll end up with

Claude Code (VS Code extension): generates diffs on a feature branch and opens a PR.

CI checks (tests/linters/security) run automatically.

Codex (VS Code extension, Agent: Full Access): reads the PR + CI results and auto-approves/merges if your policy is satisfied.

You can tighten/loosen safety with allow/deny lists and explicit prompts.

Why this shape? It avoids “UI click” approval loops and turns “approve my changes” into a Git workflow (branches/PRs/CI), which both tools handle reliably today. Claude Code’s autonomy + sandboxing and Codex’s full-access agent fit naturally here. 
Claude Code
+2
Anthropic
+2
 
OpenAI Developers
+2
OpenAI
+2

1) Install the VS Code extensions

Open VS Code → Extensions (Ctrl/Cmd+Shift+X).

Install Claude Code for VS Code by Anthropic. Sign in to Anthropic when prompted. 
Visual Studio Marketplace
+1

Install Codex — OpenAI’s coding agent (official OpenAI publisher). Sign in to your OpenAI account when prompted. In the Codex panel, choose Agent (Full Access) mode. 
OpenAI Developers
+2
DEV Community
+2

Notes
• VS Code added a unified “agent mode” experience this fall; both Anthropic and OpenAI agents slot into it well. 
Visual Studio Code

• “Full Access” lets Codex read/modify files and run commands. Some users still see prompts depending on workspace trust and extension settings—so we’ll route approvals through Git instead of UI prompts. 
OpenAI Developer Community
+1

2) Prep your repo for safe automation

From your project root:

git init
git branch -M main
git remote add origin <YOUR_GITHUB_REPO_URL>


Add CI + policy scaffolding:

Tests/linters (pytest, go test, jest, ruff, eslint… as appropriate).

Branch protection on main (require CI to pass before merge).

Optional: add Claude Code PR security review GitHub Action (it auto-comments on PRs). 
Claude

Example .github/workflows/ci.yml (skeleton; swap in your stack):

name: CI
on:
  pull_request:
    branches: [ main ]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm run lint
      - run: npm test -- --ci


(If you want Claude Code’s PR security reviewer, follow the action’s README and add a second job; see Anthropic’s blog.) 
Claude

3) Set up Claude Code to propose changes via PRs

Inside VS Code:

Open your repo → Claude Code panel.

In its settings, pick the workspace root and (optionally) enable its sandbox / path controls so edits are limited to your repo and network calls are scoped. This lets it operate more autonomously while staying safe. 
Anthropic

Give Claude a task prompt like:

“Create feature X. Work on a new branch. Make small, reviewable commits. Open a PR titled ‘feat: X’ with a summary of changes and a test plan.”

Claude will: create feat/x-..., edit files, run your dev commands, commit, and open a PR.

Why no “permission popups”?
Claude writes to your working tree/branch and uses Git—no UI approvals required. Risky ops are still gated by your sandbox and CI. 
Anthropic

4) Configure Codex (Full Access) to review & merge

In VS Code → Codex panel:

Ensure Agent Mode = Full Access. (Codex can read files, run shell, call Git, etc.) 
OpenAI Developers

Give Codex a standing reviewer prompt (save it in your workspace as .codex/reviewer.md and paste into the session at startup):

You are the Reviewer/Integrator bot.
Policy to auto-approve a PR and merge to main ONLY IF:
- CI checks are all green.
- Lint passes with no new warnings beyond baseline.
- Tests pass and coverage does not drop by >0.5%.
- No changes outside allowed paths: src/**, tests/**, docs/**
- No high-risk ops (package manager lockfile rollbacks, disabling auth/logging, network policy loosening).

Steps:
1) Fetch latest and list open PRs.
2) For the PR assigned to you, pull the branch locally.
3) Run linters/tests (same commands as CI for parity).
4) Parse CI status from GitHub API (`gh pr view --json statusCheckRollup`).
5) If policy satisfied, merge with rebase and delete branch. Else, comment blocking reasons.
Never bypass branch protections.


Ask Codex to implement a simple VS Code Task that runs the above flow with gh (GitHub CLI). It will create a script (e.g., tools/review_and_merge.sh) and a tasks.json entry so you can trigger it—or have Codex trigger it itself.

This avoids any “Codex clicks approve” UI magic. It’s normal Git + CI + gh automation under Codex’s control.

5) (Optional) Make Claude even more autonomous

In Claude Code settings, you can increase autonomy. Anthropic’s recent posts describe checkpoints and a more autonomous terminal workflow; the VS Code extension streams edits with a native GUI. 
Anthropic
+1

If you truly want “no prompts,” Anthropic’s community mentions a --dangerously-skip-permissions flag for CLI flows. Use with extreme caution and only inside a tight sandbox (devcontainer). (Community tip, not official advice.) 
Reddit

6) Run the whole loop (your first task)

In VS Code, open the repo.

Claude Code: “Implement feature Y with tests; open a PR.”

Wait for CI to finish (green).

Codex: “Review open PRs per policy in .codex/reviewer.md and merge compliant ones.”

Codex runs the task/script, verifies, merges, and deletes the branch.

That’s it—you now have Claude proposing and Codex approving without you clicking approvals.

FAQ quick hits

Q: Can Codex auto-approve Claude’s IDE prompts directly?
Not reliably across all “approve edit” UIs. Treat approvals as Git/PR decisions; Codex uses CLI + GitHub API to approve/merge once policy passes. This is sturdier and auditable.

Q: Chicken-and-egg—how does Claude write “instructions JSON” without permission?
It doesn’t need special permission to write files in your repo (it’s just editing code/JSON locally). The only “permission” that matters is merging to protected branches—which we hand to CI + Codex’s policy. 
Claude Code

Q: Do I need to weaken safety?
No. Keep: sandboxed paths, branch protections, mandatory CI. Only consider “skip prompts” inside a devcontainer for throwaway sandboxes. 
Anthropic

References & install pages

Claude Code VS Code extension (install/docs). 
Visual Studio Marketplace
+1

Anthropic: autonomy + sandboxing posts. 
Anthropic
+1

OpenAI Codex IDE extension + “Agent (Full Access)”. 
OpenAI Developers
+1

VS Code blog: unified agent experience (why both play nicely). 
Visual Studio Code

Claude Code PR security review (GitHub Action). 
Claude