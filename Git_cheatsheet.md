# Github Cheatsheet

  
## Clone
Download a repository from github into local computer
```bash
git clone https://github.com/user/repo.git
```
  
## Branch
A parallel version of the repo.
Every repo have a main branch, creating additional branch (feature branch) allows copying of main 
branch of the repo and making safe changes without disrupting main codebase

```bash
git checkout -b feature/change-A
```
- checkout: switches to diff branch, something like cd
- -b: tells Git to create new branch

## Pull
When there are updates in the main repo from github, git pull allows us to sync the updated repo with our own local repo.
Need to switch to main branch before doing the pull.
```bash
git checkout main
git pull origin main
```

Now, we can merge the updated main branch into the feature branch
```bash
git checkout feature/change-A
git merge main
```
or
```bash
git rebase main
```
- merge: bring in updates with a merge commit
- rebase: reapplies changes on top of the latest main (cleaner history)

## Commit
Set of changes to the files and folders in the project, exists inside a branch, something like a 'save point'
Commit record changes such as adding, removing, renaming files, modifying content. Commit allows a commit message to describe changes.

After done with changes A, can commit them by:
```bash
git add .                                  # stage all changes in the current folder
git commit -m "Implement changes A"        # save changes with a message
```


## Push
Send this current local branch into the same branch on github. Basically just making it online.
```bash
git push origin feature/change-A
```
- This creates or updates the feature/change-A branch on GitHub.
- It does not affect main until you open and merge a pull request.

## Pull request
A github feature to propose code changes for review and merging of the side branch into the main branch.
Ask if code can be merged into main. Reviewers will then decide to approve or merge.

## 📄 .gitignore

A `.gitignore` file tells Git to **ignore certain files or folders**, so they won’t be tracked or committed.

### ✅ Why some files should not be tracked:

1. **Temporary or generated files**  
   e.g., `__pycache__/`, `*.pyc`, `node_modules/`  
   → These are recreated automatically by your tools and just clutter the repo.

2. **Sensitive data**  
   e.g., `.env`, `secrets.json`  
   → These may contain passwords or API keys that shouldn't be shared.

3. **Large files or datasets**  
   e.g., `*.zip`, `*.mp4`, `data.csv`  
   → Git is not optimized for big files. Use external storage if needed.

4. **User or system-specific files**  
   e.g., `.vscode/`, `.DS_Store`, `Thumbs.db`  
   → These are personal and don't belong in the shared project.

5. **Keep your repo clean**  
   → Ignoring these files makes `git status` and commit history easier to manage.

### ✏️ Example `.gitignore` file:

```gitignore
# Byte-compiled files
__pycache__/
*.pyc

# Environment
.env
venv/

# Editors
.vscode/
.idea/

# System
.DS_Store
Thumbs.db
```

---

