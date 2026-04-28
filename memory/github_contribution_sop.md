# GitHub Contribution SOP

**When to use**: Whenever you plan to open a PR to an open source project (bugfix / feature / docs).

**When NOT to use**: When you are only reading code and will not submit any change.

**Core principles**
- One PR does one thing.
- Tests must pass before pushing.
- Respect the project’s conventions and maintainers.

---
## Initial setup for a new project (run once per project)

1. **Read the project’s contribution rules (mandatory)**

   Look for and read these files if they exist:

   ```
   CONTRIBUTING.md                 # Contribution guide
   .github/PULL_REQUEST_TEMPLATE.md  # PR template
   .github/ISSUE_TEMPLATE/          # Issue templates directory
   ```

   If none of these exist, read the `Contributing` / `How to contribute` section in `README`. If there is still nothing, follow this SOP as the default.

2. **Understand project structure and test commands**

   Find how to run tests (examples for common ecosystems):

   ```
   package.json      # Node: look at scripts.test
   Makefile          # Make-based workflows
   pyproject.toml    # Python: pytest / other test tools
   ```

   Write down the correct test command(s). A PR that cannot run tests locally is an unverified PR.

3. **Fork and clone**

   Typical flow using GitHub CLI (adjust OWNER/REPO):

   ```bash
   gh repo fork OWNER/REPO --clone
   cd REPO
   git remote -v
   ```

---
## Standard workflow (for each PR)

### Step 1: Clarify the goal
- Read the related issue, if any.
- Write a one-sentence description for yourself: what you are changing and why.
- Check whether someone is already working on it (issue assignee, existing PRs).

### Step 2: Create a branch

Use a descriptive branch name:

```bash
# examples
git checkout -b fix/issue-short-description
# or
git checkout -b feat/new-feature-name
# or
git checkout -b docs/update-readme
```

Naming rules:
- `fix/xxx` for bugfixes
- `feat/xxx` for new features
- `docs/xxx` for documentation-only changes

### Step 3: Implement the change

- **Keep the change set minimal**
  - Only modify what is necessary for this issue / feature.
  - Do not perform unrelated refactors or style cleanups in the same PR.

- **Match existing style**
  - Follow the project’s indentation, naming, comments, and file organization.

- **Commit frequently, with meaningful messages**

  ```bash
  git add -A
  git commit -m "fix: short, specific description"
  ```

- **Commit message format**
  - If the project defines a convention (e.g., Conventional Commits), follow it.
  - If not, use `type: short description`.
  - Common `type` values: `fix`, `feat`, `docs`, `refactor`, `test`, `chore`.

### Step 4: Run tests (must not be skipped)

Run the project’s test (and lint) commands, for example:

```bash
# examples, choose what matches the project
npm test
pnpm test
yarn test
pytest
go test ./...
# plus any lint / typecheck commands, e.g.
npm run lint
npm run typecheck
```

Checklist before pushing:
- [ ] All existing tests pass.
- [ ] New functionality has tests, if the project normally has tests for such changes.
- [ ] Linting and type checks pass, if applicable.

If tests do not pass, **do not** push. Fix the issues and rerun tests until they pass.

### Step 5: Push and open a PR

Push the branch to your fork:

```bash
git push origin HEAD
```

When creating the PR:
- **Title**
  - Follow the project’s template if present, or use `type: short description`.
- **Description** should clearly include:
  - **What**: What you changed.
  - **Why**: Why the change is needed. Link issues with `Fixes #123` / `Closes #123`.
  - **How tested**: Which commands you ran and key scenarios covered.

Avoid:
- Overly long background stories that are not relevant to the change.
- Self-promotion.

### Step 6: CI checks

After opening the PR, wait for CI results:

- ✅ All checks green → ready for review.
- ❌ Some checks failing → open the CI logs and diagnose.
  - If the failure is clearly caused by your changes, fix and push updates.
  - If CI is failing on `main` / for unrelated reasons, leave a short explanation in the PR (and optionally link to the failing workflow or upstream issue).

If using GitHub CLI, you can inspect runs like:

```bash
gh run list
gh run view --log-failed
```

### Step 7: Respond to review

- Treat reviewer feedback with respect.
  - If a reviewer asks for changes that are style-related and consistent with the project, just apply them.
- For technical disagreements:
  - Politely explain your reasoning.
  - Be prepared to accept the maintainer’s final decision.

After you make changes requested in review:
- Add new commits rather than force-pushing, unless maintainers explicitly ask for squashing or rebasing.
- Rerun relevant tests locally before pushing.
- If the reviewer requests additional tests, add them. Treat this as mandatory.

---
## Common mistakes and better alternatives

| Mistake                                      | Better practice                                  |
|---------------------------------------------|--------------------------------------------------|
| One PR changes many unrelated things        | Split into several focused PRs                   |
| Opening a PR and then ignoring feedback     | Check PR status regularly and respond promptly   |
| Pushing without running tests               | Always complete Step 4 before pushing            |
| Inconsistent coding style vs existing code  | Match the existing project style                 |
| Vague commit messages like "update"        | Use specific messages describing the change      |
| Force-pushing over reviewed history         | Prefer additional commits; squash only when asked|
| Empty or unhelpful PR description           | Always describe What / Why / How tested          |

---
## Simple lifecycle / mental model

Think of a PR’s life like this:

1. Draft changes locally.
2. Run tests and linters until they pass.
3. Push and open a PR.
4. Wait for CI.
   - If CI fails → fix and repeat.
5. Address reviewer comments.
6. Wait for merge.

During this process, use your usual tools (GitHub UI, GitHub CLI, etc.) to monitor PR status, checks, and comments.
