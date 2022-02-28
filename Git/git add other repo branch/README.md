# git add other repo branch

Created: February 1, 2022 5:22 PM
Tags: Git

git add other repo branch

```bash
git clone https://github.com/username/new_repo.git
cd fb7_re3/
git remote add old_repo https://github.com/username/old_repo.git
git fetch old_repo
git branch -a
git checkout old_repo_branch
git branch new_repo_branch
git checkout new_repo_branch
git push origin new_repo_branch
```

local change repo set new origin

```python
git remote set-url origin https://git-repo/new-repository.git

# change old branch to new branch
vim .git/config
```

[https://devconnected.com/how-to-change-git-remote-origin/](https://devconnected.com/how-to-change-git-remote-origin/)