# GitHub Actions Live Demo

A minimal Vite + React TypeScript app with a checklist UI to guide a GitHub Actions demo. Includes a test suite (Vitest + Testing Library).

## Demo Plan

1. Create CI workflow manually (run tests on push/PR)
2. Add Dependency Review using Marketplace action
3. Push code to trigger workflows
4. Inspect Actions runs and debug with logs
5. Add Azure deployment workflow (Marketplace)
6. Push again and observe end-to-end automation

## Local Usage

```bash
npm install
npm run dev
# open http://localhost:5173

npm run test         # run test suite
npm run test:watch   # watch mode
npm run build && npm run preview # preview production build
```

## GitHub Actions

- CI: `.github/workflows/ci.yml` — Node setup + install + tests
- Dependency Review: `.github/workflows/dependency-review.yml` — uses `github/dependency-review-action`
- Azure Deploy: `.github/workflows/azure-static-web-apps.yml` — deploy via Azure Static Web Apps action

### Azure Setup

- Create a Static Web App in Azure portal
- Obtain the deployment token and add repo secret: `AZURE_STATIC_WEB_APPS_API_TOKEN`
- Adjust `app_location`/`output_location` in workflow if you change paths

### Presenter Flow Script

- Create CI workflow live, commit, push → shows test run
- Add dependency-review workflow via Marketplace, commit, push PR → shows review
- Introduce a risky dependency version (e.g., `minimist@0.0.8`), push PR → action flags
- Open logs in Actions, fix dependency to safe version, push → green
- Configure Azure secret, push → deployment succeeds; open URL

## Notes

- Built on macOS, Node 18+ recommended
- Keep the demo branch small and focused
