# Amending the most recent commit message
```
git commit --amend
```
will open your editor, allowing you to change the commit message of the most recent commit. Additionally, you can set the commit message directly in the command line with:
```
git commit --amend -m "New commit message"
```
…however, this can make multi-line commit messages or small corrections more cumbersome to enter.

Make sure you don't have any working copy changes staged before doing this or they will get committed too. (Unstaged changes will not get committed.)
