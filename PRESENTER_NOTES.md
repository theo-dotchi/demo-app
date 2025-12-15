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
> "Now every pull request will automatically run tests. This prevents broken code from being merged. Let's add one more workflow using the Marketplace."

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

## Act 4: Create a Pull Request (2-3 min)

**In GitHub or terminal:**

You've already prepared a `feature-update` branch with:
1. Updated `package.json` to add `minimist@0.0.8` (intentionally risky version)
2. Updated `src/ui/App.tsx` with celebration section:
   ```tsx
   <section className="celebration">
     <span className="fireworks">🎆✨🎇</span>
     <h2>Congrats, we automatically tested and deployed our app on Azure !</h2>
     <span className="fireworks">🎇✨🎆</span>
   </section>
   ```

**Create the PR:**
```bash
# Push feature-update branch (already done)
git push origin feature-update
```

Then on GitHub:
1. Click **Compare & Pull Request** (or go to Pull requests → New)
2. Title: `"feat: add celebration message and new dependency"`
3. Base: `main` ← Compare: `feature-update`
4. Click **Create Pull Request**

**Say:**
> "Now I'm opening a pull request that adds a feature with a dependency that has known vulnerabilities. Watch what happens — the workflows will automatically check our code before it can be merged."

---

## Act 5: Watch Workflows Run on PR (5-10 min)

**Go back to the GitHub PR page:**
1. Scroll down in the PR description
2. You should see a "Checks" section with workflow runs (~30-60 seconds to start)
3. Click **Details** to see logs

**Explain what's happening:**
- **CI workflow:** ✅ **Passes** — Tests run and pass
- **Dependency Review:** ❌ **Fails** — Detects minimist@0.0.8 vulnerability

**Show the failure details:**
1. Click the failed **Dependency Review** check
2. Expand the logs
3. Read the advisory message explaining the vulnerability in minimist 0.0.8

**Say:**
> "The PR-triggered workflows immediately caught a high-severity vulnerability in our new dependency. This is exactly what we want — automation preventing insecure code from being merged into production. Developers get instant feedback without waiting for manual review."

---

## Act 6: Ask Copilot to Explain the Failure (2-3 min)

**In VS Code / Terminal:**

Update the dependency to a safe version:
```bash
npm install minimist@1.2.8
git add package.json package-lock.json
git commit -m "fix: update minimist to safe version"
git push
```

2. The PR auto-updates, workflows re-run

**Watch it turn green:**
1. Go back to PR page
2. Refresh and wait for new workflow runs (~30-60 sec)
3. Both CI ✅ and Dependency Review ✅ now pass

**Say:**
> "Now that we've fixed the vulnerability, the workflows are green. The PR is safe to merge. This is the full automation loop — detect, fix, verify, merge."

---

## Act 7: Deploy (Optional / Time Permitting)

**If time allows:**

Create the Azure deployment workflow manually:

**Add file:** `.github/workflows/azure-deploy.yml`

```yaml
name: Deploy to Azure

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Build
        run: npm ci && npm run build

      - name: Deploy to Azure
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_ASHY_PEBBLE_054BD1210 }}
          action: "upload"
          app_location: "/"
          output_location: "dist"
```

**Explain:**
> "This workflow automatically deploys our built app to Azure Static Web Apps whenever we push to the main branch. The secret token is stored securely in GitHub and used during deployment."

**Push and show the deployment workflow running.**

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
