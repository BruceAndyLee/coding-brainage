---
Status: Completed
tags:
  - infra
  - vcs
---
Learn: [oh my git](https://ohmygit.org)

---

```Bash
# Reverting a merge commit
git revert -m 1 <merge_commit_sha>
```

```Bash
# разрешение конфликтов
git checkout --theirs path/to/file
git checkout --ours path/to/file
```

```Bash
# file specific difference
git diff mybranch..master -- myfile.cs
```

```Bash
# creating a patch file
git diff branch_name -- [path/to/file] > patch_file_name.patch

# for staged changes:
git diff --staged branch_name -- [path/to/file] > patch_file_name.patch
```

```Bash
# applying a patch file
git apply patch_file_name.patch
```

очистка кеша [https://gist.github.com/pavankjadda/2bb6fbdd8786e1f57fd7bcbcc666b51d](https://gist.github.com/pavankjadda/2bb6fbdd8786e1f57fd7bcbcc666b51d)

```bash
# logging a history of changes for a specific range of lines
git log -L 1,34:path/to/file.txt
```

```bash
# find an entrance in the history of the project to see when a string was removed
git log -S <regex> 
```

```bash
# find an string entry in the scope of the project
git grep <some-string>
```

```bash
git branch --column # to tell the git branch output to try and fit branches into multiple columns on the page
git config --global column.ui auto

# to sort the branch-command output by the commit date
# to put first the branches with most recent commits
git config --global branch.sort -comitterdate
```

```bash
# show history as a graph
git log --graph
git log --graph --oneline # only show one line per branch?
git config --global fetch.writeCommitGraph true # to tell git to rebuild and re-cache the graph on each fetch
```

```bash
# to speed up the FS-analysis for "git status" commands in the big projects thanks to a daemon:

git config core.fsmonitor true
```