---
layout: post
category: "工具"
title:  "git开发流程"
---

1.根据任务从dev分支新建一个分支task1 
git checkout develop
git checkout -b task1
coding...
git commit -am "task"

2.测试
git checkout test
git merge task1
git push

3.上线
git checkout develop
git merge task1
git push

4.删除任务分支
git branch -d task1

紧急
跟上面一样新建一个bug分支，但是bug分支不用每次都新建，可以定期拉一次（保证和其他分支同步）
git checkout develop
git checkout -b bug
coding...
git commit -am "bug"

git checkout test
git merge bug
git push

git checkout develop
git merge bug
git push