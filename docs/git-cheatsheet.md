# Git Cheatsheet

## Sync Fork With Upstream

Use this when you want your fork to catch up with the parent repository.

```powershell
git checkout main
git fetch upstream main
git merge upstream/main
git push origin main
```

## Check Remotes

Verify that both your fork and the parent repository are configured.

```powershell
git remote -v
```

Expected pattern:

- `origin` -> your fork
- `upstream` -> parent repository

## Add Upstream Remote

Run this once if `upstream` does not exist.

```powershell
git remote add upstream https://github.com/github/awesome-copilot.git
```

## Check Current Status

Use this before and after syncing.

```powershell
git status
```

## Handle Merge Conflicts

If the merge conflicts and you want to accept the upstream version of generated files:

```powershell
git checkout --theirs docs/README.instructions.md docs/README.skills.md
git rm docs/README.prompts.md
git add docs/README.instructions.md docs/README.skills.md
git commit -m "Sync with upstream: resolve merge conflicts"
git push origin main
```

## Start New Work After Sync

Create a feature branch from the updated `main` branch.

```powershell
git checkout main
git pull origin main
git checkout -b my-feature-branch
```

## Useful Inspection Commands

See commits you have locally that are not yet on your fork:

```powershell
git log origin/main..HEAD --oneline
```

See what changed between your branch and upstream:

```powershell
git log HEAD..upstream/main --oneline
git diff HEAD..upstream/main
```

## Abort A Merge

If a merge goes wrong and you want to stop before committing:

```powershell
git merge --abort
```

## Recommended Routine

```powershell
git checkout main
git fetch upstream main
git merge upstream/main
git push origin main
git checkout -b feature/something-new
```