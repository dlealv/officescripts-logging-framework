# Git Basic Operations Reference

A concise cheat sheet with essential commands and common workflows for daily Git usage.  
**Tip:** Use `git help <command>` for detailed info about any command!

---

## 1. Setup

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global core.editor "code --wait"  # Set VSCode as editor (optional)
```

---

## 2. Creating a Repository

```bash
git init                # Initialize new repo in current directory
git clone <url>         # Clone an existing repository
```

---

## 3. Checking Status

```bash
git status              # Show status of changes
```

---

## 4. Staging and Committing

- **Staging**: Moves changes from your working directory to the staging area.
- **Committing**: Captures a snapshot of the currently staged changes.

```bash
git add <file>                              # Stage a specific file
git add .                                   # Stage all files (including new, modified, deleted)
git commit -m "Message"                     # Commit staged changes
git commit --allow-empty -m "Message"       # Commit with no changes (create marker)
```

---

## 5. Viewing History

```bash
git log                      # View commit history
git log --oneline            # Condensed log (one commit per line)
git log -p <file>            # Show changes for a file over time
git diff <commit> <commit>   # Show differences between two commits
```

---

## 6. Working with Branches

```bash
git branch                   # List branches
git branch <name>            # Create new branch
git checkout <name>          # Switch to branch
git checkout -b <name>       # Create and switch to branch
git merge <name>             # Merge branch into current
git branch -d <name>         # Delete a branch
git branch -m <old> <new>    # Rename a branch
```

---

## 7. Pulling and Pushing

- **Pulling**: Fetches changes from a remote repository and integrates them into your local branch.
- **Pushing**: Uploads local commits to a remote repository.

```bash
git pull                      # Fetch and merge from remote (default remote/branch)
git pull origin main          # Fetch and merge changes from 'main' on 'origin'
git push                      # Push changes to remote (current branch)
git push -u origin <branch>   # Push new branch and set upstream tracking
```
**Note**: `origin` is the default name of the remote repository on GitHub.

---

## 8. Undoing Changes

```bash
git checkout -- <file>        # Discard local changes in working directory (before staging)
git reset HEAD <file>         # Unstage a file (keep changes in working directory)
git revert <commit>           # Create new commit to undo changes (safe, doesn't rewrite history)
git reset --hard <commit>     # Danger! Discard ALL history and changes since <commit>
```

---

## 9. Tags

```bash
git tag                       # List tags
git tag <name>                # Create tag at current commit
git tag -a <name> -m "msg"    # Annotated tag with message
git push origin <tag>         # Push tag to remote
git push origin --tags        # Push all tags to remote
```

---

## 10. Rebasing (Advanced)

- Modify the commit history of a branch by moving or combining a sequence of commits to a new base commit.

```bash
git rebase <base-branch>          # Rebase current branch onto base
git rebase -i <commit-hash>       # Interactive rebase for editing history
```

---

## 11. Stashing

- Temporarily save uncommitted changes.

```bash
git stash                         # Stash unsaved changes
git stash pop                     # Apply and remove latest stash
git stash list                    # List all stashes
git stash apply stash@{n}         # Apply specific stash (keeps it in stash list)
```

---

## 12. Common Multi-Step Workflows

### A. Sync local repo with remote (bring latest changes)

```bash
git checkout main
git pull origin main
```

### B. Create and work on a new feature branch

```bash
git checkout main
git pull origin main            # Make sure main is up to date
git checkout -b my-feature      # Create and switch to new branch
# ...edit files...
git add .
git commit -m "Implement feature"
git push -u origin my-feature   # Push branch and set upstream
```

### C. Merge a finished feature branch into main

```bash
git checkout main
git pull origin main            # Update local main
git merge my-feature
git push origin main            # Push merged changes
```

### D. Update your feature branch with latest main

```bash
git checkout my-feature
git pull origin main            # Merge latest main into your branch
```
*(Or use `git rebase origin/main` if rebasing is preferred)*

### E. Delete a merged feature branch (locally and remotely)

```bash
git branch -d my-feature        # Delete local branch
git push origin --delete my-feature   # Delete remote branch
```

---

**Tip:**  
Use `git help <command>` for detailed info about any command!