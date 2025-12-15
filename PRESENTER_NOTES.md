# GitHub Actions Live Demo — Presenter Notes

## Overview
This is a training/interview demo showing GitHub Actions for CI/CD automation. The demo repo has a React + TypeScript app with tests, and you'll create workflows live to show automation.

---

## Act 1: The Application (2-3 min)

**Show the running app** at `http://localhost:5173`

**Say:**
> "This is a simple React + TypeScript application built with Vite. It demonstrates the checklist of what we'll do today. But the real demo is about automation — how we can use GitHub Actions to run tests, check dependencies, and deploy automatically on every push."

**Point out:**
- The app has a test suite (Vitest)
- Clean, simple UI showing the demo flow
- Running locally for now

**Transition:** "Let's go to GitHub and create the workflows."

---

## Act 2: Create CI Workflow Manually (5-7 min)

**Navigate to GitHub:**
1. Go to https://github.com/theo-dotchi/demo-app (on `main` branch)
2. Click **Add file** → **Create new file**
3. Path: `.github/workflows/ci.yml`

**Explain the structure:**
> "GitHub Actions workflows are YAML files in the `.github/workflows` folder. Each workflow defines jobs and steps that run automatically on events like push or pull request."

**Type/paste the CI workflow:**
```yaml
name: CI

on:
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm run test
```

**Explain each section:**
- **`on:`** — When the workflow triggers (push, PR)
- **`jobs:`** — What to run
- **`runs-on: ubuntu-latest`** — The runner environment
- **`uses: actions/checkout@v4`** — First step: get the code (from GitHub's official action)
- **`uses: actions/setup-node@v4`** — Second step: set up Node (cached npm for speed)
- **`run: npm ci`** — Install exact dependency versions
- **`run: npm run test`** — Execute tests

**Show the action repo:**
> "These `uses:` lines reference actions maintained by GitHub. Let me show you — I can click on `actions/checkout` and it opens the official repo where this action lives."

Open https://github.com/actions/checkout in a new tab for 10 seconds, then go back.

**Commit the file:**
- Message: `"ci: add workflow to run tests"`
- Commit directly to `main`

**Say:**
> "Now every push will automatically run tests. Let's add one more workflow using the Marketplace."

---

## Act 3: Add Dependency Review from Marketplace (3-5 min)

**Go to Actions tab:**
1. Click **Actions** tab on the repo
2. Click **New workflow** (or search in Marketplace)
3. Search: `Dependency Review`
4. Find `github/dependency-review-action` (official GitHub action)
5. Click **Configure**

**Explain:**
> "Instead of writing this from scratch, I'm using a pre-built action from the GitHub Marketplace. This one automatically checks for known vulnerabilities in dependencies when you open a pull request."

**The workflow auto-generates.** Review the file:
```yaml
name: Dependency Review

on:
  pull_request:
    branches: [ main ]

permissions:
  contents: read

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Dependency review
        uses: github/dependency-review-action@v4
```

**Say:**
> "It's pre-configured to fail on high-severity issues. This will catch risky dependencies automatically. Let's commit it."

**Commit the file:**
- Message: `"ci: add dependency review action"`

---

## Act 4: Push Code Changes (2-3 min)

**In VS Code (or terminal):**

Show that you've already:
1. Added a new dependency: `minimist@0.0.8` (intentionally risky version)
2. Made a small code change (e.g., added a feature)

**Run:**
```bash
git add .
git commit -m "feat: add feature with new dependency"
git push
```

**Say:**
> "Now I'm pushing a change that adds a dependency with known vulnerabilities. Watch what happens when this triggers the workflows..."

---

## Act 5: Watch Workflows Run (5-10 min)

**Go back to GitHub:**
1. Refresh the repo
2. You should see a new commit on `main`
3. Click **Actions** tab
4. Watch both workflows execute (~30-60 seconds)

**Explain what's happening:**
- **CI workflow:** ✅ **Passes** — Tests are green
- **Dependency Review:** ❌ **Fails** — Detects minimist@0.0.8 vulnerability

**Show the failure details:**
1. Click the failed **Dependency Review** run
2. Expand the logs
3. Read the advisory message explaining why it failed

**Say:**
> "The workflow detected a known high-severity vulnerability in minimist 0.0.8. This is exactly what we want — automation catching security issues before they go live."

---

## Act 6: Fix and Re-run (3-5 min)

**In VS Code / Terminal:**

**Ask Copilot to explain the failure:**
- Open the failed Dependency Review workflow run and copy the advisory text.
- In Copilot Chat, ask: "Explain why this workflow failed and how to fix it." 
- Copilot responds with guidance to upgrade `minimist` (e.g., `>=1.2.6`) and suggests running `npm audit` to check for additional issues.

**Note to audience (why only `minimist` was flagged):**
- Dependency Review scans changes in the PR and flags vulnerabilities introduced by the dependencies you added or updated.
- It does not report existing or unrelated transitive/dev vulnerabilities that were already in the base branch.
- `npm audit` reports all known issues in the whole dependency tree (including dev and transitive), which is why it may show more findings than Dependency Review.

Update the dependency to a safe version:
```bash
npm install minimist@1.2.8
git add package.json package-lock.json
git commit -m "fix: update minimist to safe version"
git push
```

**Say:**
> "Now let's fix the vulnerability and push again. The workflow will automatically re-run."

**Watch it turn green:**
1. Go back to **Actions** tab
2. Watch the new run (CI passes again, Dependency Review now passes ✅)

**Say:**
> "Both workflows are now green. Everything is clean. If this were production, we could now safely merge and deploy."

---

## Act 7: Deploy via Marketplace (Optional / Time Permitting)

**Goal:** Use the official Marketplace template instead of writing YAML by hand.

**Steps (in GitHub UI):**
1. Go to the **Actions** tab → Click **New workflow**.
2. Search for and select: **Azure Static Web Apps** (by Microsoft).
3. Click **Configure**.
4. In the generated workflow form, set:
   - App location: `/`
   - API location: leave blank (not using an API)
   - Output location: `dist`
5. Ensure the workflow targets your deploy branch (usually `main`).
6. If the repo isn’t already connected from the Azure portal, add the required secret:
   - `AZURE_STATIC_WEB_APPS_API_TOKEN` (or the resource-specific token Azure creates automatically for the Static Web App).
7. Commit the generated workflow (filename will look like `azure-static-web-apps-<suffix>.yml`).

**What this template does:**
- Uses `Azure/static-web-apps-deploy@v1` with Azure’s integrated build (no manual `npm ci`/`npm run build` steps needed).
- On pull requests to `main`, it creates a preview environment and tears it down on PR close.
- On push to `main`, it builds and deploys to production.

**Say:**
> "Using the Marketplace template is faster and includes best practices like preview environments. When I push to `main`, it builds and deploys automatically using the repo secret."

**Trigger it:** Merge your PR to `main` (or push to `main`) and watch the **Azure Static Web Apps** workflow run.

---

## Key Talking Points

- ✅ **Workflows are code:** YAML files, versioned, reviewed in PRs
- ✅ **Actions are reusable:** Pre-built by GitHub and community
- ✅ **Automation catches issues:** Dependencies, tests, deployments all automated
- ✅ **Transparent logs:** Every step is visible, easy to debug
- ✅ **Marketplace:** Thousands of ready-to-use actions for any use case

---

## Timing

- App demo: 2-3 min
- Create CI: 5-7 min
- Add Dependency Review: 3-5 min
- Push code: 2-3 min
- Watch workflows: 5-10 min
- Fix & re-run: 3-5 min
- Deploy (optional): 3-5 min

**Total: 15-20 minutes** ✓

---

## Quick Troubleshooting

| Issue | Fix |
|-------|-----|
| Workflows don't run | Check Actions tab; they may take 30 sec to start |
| Dependency Review doesn't fail | Make sure `minimist@0.0.8` is in `package.json` |
| Tests fail | Verify `npm run test` works locally first |
| Confused about YAML? | Blame spaces/indentation; YAML is strict |

---

## Questions You Might Get

**Q: How do I know which actions to use?**  
A: GitHub Marketplace (https://github.com/marketplace) has thousands. Start with official GitHub actions, then community-maintained ones.

**Q: Can I use my own Docker image?**  
A: Yes! `docker://` syntax. But actions are simpler for most tasks.

**Q: What if a workflow fails in production?**  
A: Logs are always visible. You can re-run jobs, view artifacts, and debug step-by-step.

**Q: How do I keep workflows DRY (don't repeat yourself)?**  
A: Use reusable workflows and composite actions. We can discuss that in a follow-up.

---

**Good luck! You've got this.** 🚀
