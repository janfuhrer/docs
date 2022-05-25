# docs: git/snippets
#git #snippets 
## sign last N commits
```bash
git rebase HEAD~3 --signoff
```

## revert commit
If you have commit, but not pushed:

```bash
git reset --soft HEAD~1
```

If you commit and push into repository:
- This will create a new commit that reverses everything introduced by the accidental commit.

```bash 
git revert HEAD
```

## log
```bash
# log with diff
git log -p

# one line
git log --oneline
```

## conflicts
1. pull each branch
```bash
git fetch #optional

git checkout dev
git pull

git checkout master
git pull
```
2. Checkout branch you would merge into (in this case master) and merge
```bash
git checkout master
git merge dev
```
3. Now it shows conflicts, and write both versions in the files
```bash
git diff
```
4. Fix all conflicts (delete all conflict-lines)
5. Add changed files
```bash
# optional, shows something
git stash 

# add files
git add <files>

# show diff
git diff --staged

# show status -> all conflicts fixed, but still in merging-state
git status
```
6. Commit changes
```bash
git commit

# show log with diff
git log -p
```
7. Push changes
 ```bash
 git push 
 ```