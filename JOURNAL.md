## Week 7 — Issue selection

**Issue link:** [https://github.com/ascherj/pathreview/issues/128]

**Issue title:** [Add a dependency vulnerability scan to the CI pipeline]

**Tier:** [ ] Tier 1 [ ] Tier 2 [✓] Tier 3

**Problem summary:**
[My issue (#128) is to build an automated dependency security scan that will run when a pull request is targeted towards main and when code is pushed directly to main. What is currentely missing is a programmable function that will automate the calling of "pip audit" and "npm audit" to run the security scan on the codebase's Python and JavaScript dependences. A successful fix would look a github function that sends a fail message showing the high-priority vulnerabilities found in the codebase. The part of the codebase that it affects are the .github workflows where it starts, but will check over the entire codebase to find vulnerabilities.]

**Branch name:** [fix/128-add-a-dependency-vulnerability-scan-to-the-ci-pipeline]

**Setup confirmation:** [✓] App runs locally at localhost:5173

**Cohort ledger:** [✓] Issue added to cohort ledger

# Section Notes

## Part 1 — Understanding the Issue

Can I explain what this issue is asking for in my own words?

- The problem is that there is no automated scan for security vulnerabilities in the continuous integration pipeline. The expected behavior is that in the CI pipeline, whenever a pull request targets main and when code is pushed directly to main, the program will call pip audit and npm audit to find security flaws in the codebase.

Do I understand which part of the app is affected?

- Yes, the part of the app that is affected is the ci.yml workflow in the .github folder. This issue is strictly devops.

Do I understand what "done" looks like?

- The before is that a developer can send pull requests with no certainty of it being secure, and users can interact with a vulnerable product. After, a developer gets a breakdown of vulnerability findings which they can use to fix the code, and the users will be working with a fully secure product.

## Part 2 — Tier Fit

Is the tier a realistic match for where I am right now?

- While not on my public profile, I have contributed numerous times to a large codebase in a work environment, from tasks like simple frontend implementation to full on RESTful API building. I have also worked on CI/CD pipelines before using a database, Docker, and GitHub Actions. I believe I have the experience to take on this Tier 3.

## Part 3 — Codebase Readiness

Can I find the relevant code?

- I have found the necessary code with relative ease in .github/workflows.

Do I understand the surrounding code well enough to change it safely?

- I have read the file and surrounding context (such as the other workflow, safety/security based code, and tests) and can write a rough plan for the fix.

Have I read the relevant test file?

- I have found the test file for my module and will need to make tests for my end-to-end.

## Part 4 — Scope and Time

How many others are already working on this issue?

- I have checked the issue comments and ledger's claims counts, there are 3 other students working on this issue, none of which are currentely in my session as of right now. I am comfortable with how many others are working on this issue.

Is the scope realistic for Weeks 8–9?

- While the PR says this should only be a 3-5 hour issue, with planning, testing, and documentation, it will take longer. That being said, 5 hours + the amount of time needed to do the extras is both very reasonable in a two week timeframe, as well as giving me enough cushion time in the case that I am working slow or am stuck.

Are there any blockers or dependencies?

- My issue does not claim to have any blockers or dependencies.

## Week 8 — Reproduction

This issue is a feature gap (no dependency vulnerability scan exists), not a runtime bug, so "reproducing" it means confirming exactly what's missing and proving the gap is exploitable right now with real data.

**Where the gap lives:**

- [.github/workflows/ci.yml](.github/workflows/ci.yml) defines 5 jobs (`lint`, `typecheck`, `test-unit`, `test-integration`, `frontend`) and none of them run `pip-audit` or `npm audit`. There is no `security`/`audit` job, no `dependabot.yml`, and no vulnerability-scanning tooling anywhere in the repo (confirmed via a repo-wide grep for `audit|dependabot|trivy|snyk|safety`).
- [pyproject.toml](pyproject.toml) `[project.optional-dependencies].dev` has no `pip-audit` entry, so it isn't even installed for the `typecheck`/`test-*` jobs to piggyback on.
- Net effect: a PR that introduces a vulnerable dependency currently passes CI with no signal at all.

**Proof the gap is live (not hypothetical) — run locally today:**

1. Frontend: `cd frontend && npm audit --json`
   - Result: **11 vulnerable packages** — 1 critical (`vitest`), 5 high (`form-data`, `picomatch`, `postcss`, `vite`, `ws`), 4 moderate, 1 low. None of this surfaces anywhere in CI today (the `frontend` job only runs `npm ci` + `npm test`).
2. Backend: installed `pip-audit` into a scratch venv and ran `pip-audit .` (audits `pyproject.toml`'s resolved dependency set) from the repo root.
   - Result: **2 known vulnerabilities** — `chromadb` (`PYSEC-2026-311`) and `ecdsa` (`PYSEC-2026-1325`, pulled in transitively), exit code 1 (`pip-audit` fails the process on findings, which is exactly the signal a CI job needs to gate on).

Both commands are reproducible by anyone with Node/Python installed and require no code changes — they demonstrate the exact failure mode the issue describes: real, currently-undetected vulnerabilities flowing straight through CI. The fix is to add a `security-scan` (or similar) job to `ci.yml` that runs both commands on `pull_request`/`push` to `main` and fails the build on high/critical findings.
