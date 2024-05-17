---
title: 'git: rebase 重复提交问题'
date: 2024-05-12 10:50:38
categories: git
tags:
---

### QA

git rebase 出现重复提交的原因可能是：

有时候rebase会出现冲突，在解决冲突后可能会错误地再次使用 `git add` 命令将已经解决冲突的文件添加到暂存区，从而导致相同的提交重复应用到最终的提交历史。



解决方法：

1. 可以在解决冲突后，使用`git rebase --skip`命令跳过当前提交，避免将解决冲突的提交重复应用
2. 在进行rebase操作之前，先使用`git log`命令查看提交历史，并使用交互式rebase来删除重复的提交

### References

- https://geek-docs.com/git/git-questions/1298_git_git_commits_are_duplicated_in_the_same_branch_after_doing_a_rebase.html
