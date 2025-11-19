MAINTAINING THE REPOSITORY
===========================

This document explains a step-by-step workflow to keep your fork of `awesome-copilot` synchronized with the upstream repository while avoiding accidental overwrites or lost commits.

1. Keep a safety backup branch before risky operations
    - Before merging, rebasing, resetting, or force-pushing, create a local backup branch:

       ```bash
       git branch backup/main-YYYYMMDD
       ```

2. Keep remotes configured safely
    - `origin` should point to your fork and be pushable.

       ```bash
       git remote set-url origin https://github.com/gncube/awesome-copilot.git
       ```
    - `upstream` should point to the canonical repository and normally be fetch-only. Remove any push URL for `upstream` or set it to a non-writable placeholder to prevent accidental pushes:

       ```bash
       git remote set-url upstream https://github.com/github/awesome-copilot.git
       git remote set-url --push upstream no_push
       ```

3. Stay in sync (recommended: merge workflow)
    - Fetch upstream and merge regularly to keep your fork current while keeping your commits:

       ```bash
       git fetch upstream
       git checkout main
       git merge upstream/main
       ```
   - If merge conflicts occur, resolve them locally, test, and commit the resolution. Do not force-push unless you intentionally want to rewrite history.

4. Alternative: Rebase workflow (use with care)
    - If you prefer a linear history and are the only one working on your fork, you can rebase your `main` on top of `upstream/main`:

       ```bash
       git fetch upstream
       git checkout main
       git rebase upstream/main
       ```
   - Resolve conflicts during rebase, then continue with `git rebase --continue`. If you rebased local commits already pushed to `origin`, you will need to force-push:
   ```bash
   git push --force-with-lease origin main
   ```
   - Prefer `--force-with-lease` to reduce the risk of overwriting others' work.

5. If you want to match upstream exactly (discard your commits)
    - This operation will remove your fork's local commits. Only do this if you are sure you want to abandon them.

       ```bash
       git fetch upstream
       git checkout main
       git reset --hard upstream/main
       git push --force origin main
       ```

6. Review what GitHub will do before clicking buttons on the website
   - The GitHub web UI may suggest updating or synchronizing branches. When it warns that commits will be discarded, treat that as a red flag. Prefer merging locally where you control conflict resolution.

7. Keep `push.default` safe
    - Ensure your global or repo setting uses a conservative push default:

       ```bash
       git config --global push.default simple
       ```

8. Add safeguards
   - Consider a local `pre-push` Git hook that prevents pushing to `upstream` or prevents force pushes unless a special flag is present.

9. Routine checklist for updating your fork
   - Fetch upstream: `git fetch upstream`

   - Inspect divergence: `git rev-list --left-right --count upstream/main...main`

   - Create backup branch: `git branch backup/main-YYYYMMDD`

   - Merge: `git merge upstream/main` (or rebase if desired)

   - Run tests / linters / build if available

   - Resolve conflicts -> `git add` -> `git commit`

   - Push to origin: `git push origin main`

10. If something goes wrong
    - Restore your backup branch:
      ```bash
      git checkout main
      git reset --hard backup/main-YYYYMMDD
      git push --force-with-lease origin main
      ```

11. Communicate with upstream maintainers
    - If your fork introduces many changes and you'd like them merged upstream, open a clear PR with a summary of changes and request a review. Be prepared to trim or split changes if maintainers ask.

12. Recommended Git aliases (optional)
   ```bash
   git config --global alias.up 'fetch upstream && merge upstream/main'
   git config --global alias.backup 'branch backup/$(date +%Y%m%d)'
   ```

13. Housekeeping
    - Keep `README` and contributor docs up to date.
    - Run `npm ci`/`npm install` and any project-specific validation before pushing large merges.

By following this workflow you'll avoid accidental loss of commits and keep your fork in sync with upstream safely.
