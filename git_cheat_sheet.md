### Creating & Copying Projects
| Command <img width=500/> |  Description <img width=500/>  |  
|----------|-------------|
| git init |  Initialize a local Git repository |
| git clone ssh://git@github.com/[username]/[repository-name].git |    Create a local copy of a remote repository   |  

_____________
### Basic Snapshotting

| Command <img width=500/>  |      Description  <img width=500/>    |  
|----------|-------------|
| git status | Check status |
| git add [file-name.txt] | Add a file to the staging area | 
| git add -A | Add all new and changed files to the staging area | 
| git commit -m "[commit message]" | Commit changes | 
| git rm -r [file-name.txt] | Remove a file (or folder) | 

____________
### Branching & Merging

| Command  <img width=500/> |      Description   <img width=500/>   |  
|----------|-------------|
| git branch | List branches (the asterisk denotes the current branch) | 
| git branch -a | List all branches (loca and remote) | 
| git branch [branch name] | Create a new branch |
| git branch -d [branch name] | Delete a branch | 
| git push origin --delete [branch name] | Delete a remote branch | 
| git checkout -b [branch name] | Create a new branch and switch to it | 
| git checkout -b [branch name] origin/[branch name] | Clone a remote branch and switch to it | 
| git branch -m [old branch name] [new branch name] | Rename a local branch | 
| git checkout [branch name] | Switch to a branch | 
| git checkout - | Switch to the branch last checked out | 
| git checkout -- [file-name.txt] | Discard changes to a file | 
| git merge [branch name] | Merge a branch into the active branch | 
| git merge [source branch] [target branch] | Merge a branch into a target branch | 
| git stash | Stash changes in a dirty working directory | 
| git stash clear | Remove all stashed entries | 

_______________
### Sharing & Updating Projects
| Command  <img width=500/> |      Description  <img width=500/>    |  
|----------|-------------|
| git push origin [branch name] | Push a branch to your remote repository | 
| git push -u origin [branch name] | Push changes to remote repository (and remember the branch) | 
| git push | Push changes to remote repository (remembered branch) | 
| git push origin --delete [branch name] | Delete a remote branch |
| git pull | Update local repository to the newest commit |
| git pull origin [branch name] | Pull changes from remote repository | 
| git remote add origin ssh://git@github.com/[username]/[repository-name].git | Add a remote repository | 
| git remote set-url origin ssh://git@github.com/[username]/[repository-name].git | Set a repository's origin branch to SSH | 
| git remote -v | Shows URL's of remote repository when listing your current remote connections | 
| git remote add upstreatm `https://github.com/`[origonal owner]/[origonal repository] | Specify a new remote upstream repo that will be synced with the fork | 
| git fetch upstream | Sync a fork of a repository to keep it up-to-date with the upstream master repository | 
| git rebase upstream/master | | Rewrite master to add any commits that are not already in upstream/master |

________________
### Inspection & Comparison
| Command  <img width=500/> |      Description  <img width=500/>    |  
|----------|-------------|
| git log | View changes | 
| git log --summary | View changes (detailed) | 
| git log --oneline | View changes (brief) | 
| git diff [source branch] [target branch] | Preview changes before merging | 


____________
*Adapted from David John Gagne, NCAR-CISL*


