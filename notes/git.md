# Git

### Create New Repo
1. Create empty repo on Github
2. (Alternatively initialize an empty remote repo on a vps using ` git init --bare REMOTE_DIR `)
3. Navigate to project directory
``` bash
cd PROJECT_DIR
```
4. Initialize local repository
``` bash
git init
```
5. Add all applicable files (use * for wildcard matching)
```  bash
git add FILES
```
6. Commit the changes
``` bash
git commit -m "MESSAGE"
```
7. Link remote repo on Github
``` bash
git remote add origin URL
```
8. Set upstream tracking to Github and push the local changes
``` bash
git push -u origin master
```

### Clone Existing Repo
* Clone the remote repo into a sub direectory of the current path
``` bash
git clone URL
```
* Clone only the desired branch
``` bash
git clone --single-branch -b BRANCH_NAME REPO_NAME
```

### Branch Management
* Checkout a new branch without history
``` bash
git checkout --orphan NEW_BRANCH
```
* Delete the remote branch
``` bash
git push origin -d BRANCH_NAME
```
* Rebase old parent D onto new parent commit F
``` bash
git rebase --onto F D
```
After running rebase, Git will start adding each commit iteratively onto the base branch.
It will fast-forward each of these commits when possibly and ask for you to manually resolve conflicts when necessary.
Run ` git rebase --continue ` after manual merges to continue the rebase process.

### Merge
``` bash
git checkout master_branch
git pull -f
git checkout feature_branch
merge master_branch
# Manually merge conflicts in files
git add -u
git commit -m "Merged..."
git push
```

### Remote Merge
``` bash
git pull origin REMOTE_BRANCH
# Make local changes
git add -until
git commit
```
If merge fails to revert to last local commit (ensure local work is saved before merging)
``` bash
git merge --abort
```

### Recover Lost Commits
``` bash
git reflog show | head -10
git reset --hard HEAD@{i}
```

### Stash
``` bash
git stash
git stash pop
git stash clear
```

### Clean Old Commits
``` bash
git checkout BRANCH
git checkout --orphan temp COMMIT_HASH
git commit -m "MESSAGE"
git rebase --onto temp COMMIT_HASH BRANCH
git branch -d temp
# Alternatively, squash the last n commits
git reset --soft HEAD~n && git commit
```

### Misc
* Show nice commit history
``` bash
git log --graph --decorate --pretty=oneline --abbrev-commit
```
* Delete remote branch
``` bash
git push origin -d BRANCH_NAME
```
* Change repo origin address
``` bash
git remote set-url origin ORIGIN_URL
```
