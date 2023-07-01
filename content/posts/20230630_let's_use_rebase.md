---
title: "使用Git Rebase"
author: jasontr
date: 2023-06-30T10:34:47+09:00
---

当我们在开发时，我们经常需要在多个分支上工作。在此情况下，我们可能会遇到一个分支（比如分支 B）在另一个分支（比如分支 A）之后开始开发，但却先被合并到主分支的情况。

在这种情况下，如果我们直接尝试将分支 A 合并到主分支，很可能会遇到许多冲突。这是因为分支 A 是基于当时的主分支创建的，而那时的主分支并没有分支 B 的提交。因此，分支 A 并不知道分支 B 的改动，这就会导致合并冲突。

要解决这个问题，我们可以在合并分支 A 之前，先执行 `git rebase`。这将把分支 A 的基准 `commit` 更新为当前的主分支状态，这样分支 A 就可以获取到分支 B 的改动，从而避免合并冲突。同时让commit的graph更加的整洁

## 代码和 Mermaid 图例

1. User1在主分支上开始开发，然后创建了分支 A（commit1 是创建分支 A 的起点）。

2. User1在分支 A 上做了提交commit2 。
   ```mermaid
   gitGraph
       commit id: "commit1" tag: "Start of Branch A"
       branch A
       commit id: "commit2"
       checkout main
   ```

3. 然后User2创建了分支 B（commit4 是创建分支 B 的起点）。
   ```mermaid
   gitGraph
       commit id: "commit1" tag: "Start of Branch A"
       branch A
       commit id: "commit2"
   		checkout main
       branch B
       commit id: "commit5" tag: "Start of Branch B" type: highlight
   ```

   

4. 然后User1在分支 A 上做了提交commit3 。
   ```mermaid
   gitGraph
       commit id: "commit1" tag: "Start of Branch A"
       branch A
       commit id: "commit2"
   		checkout main
       branch B
       commit id: "commit5" tag: "Start of Branch B" type: highlight
       checkout A
       commit id: "commit3"
   ```

   

5. 然后User2在分支 B 上做了提交commit6 。
   ```mermaid
   gitGraph
       commit id: "commit1" tag: "Start of Branch A"
       branch A
       commit id: "commit2"
   		checkout main
       branch B
       commit id: "commit5" tag: "Start of Branch B" type: highlight
       checkout A
       commit id: "commit3"
       checkout B
       commit id: "commit6" type: highlight
   ```

   

6. User2先将分支 B 合并到了主分支。

   ```mermaid
   gitGraph
       commit id: "commit1" tag: "Start of Branch A"
       branch A
       commit id: "commit2"
   		checkout main
       branch B
       commit id: "commit5" tag: "Start of Branch B" type: highlight
       checkout A
       commit id: "commit3"
       checkout B
       commit id: "commit6" type: highlight
       checkout main
       merge B
   ```

   ```mermaid
   gitGraph
       commit id: "commit1" tag: "Start of Branch A"
       branch A
       commit id: "commit2"
   		checkout main
       commit id: "commit5" tag: "Start of Branch B" type: highlight
       checkout A
       commit id: "commit3"
       checkout main
       commit id: "commit6" type: highlight
   ```

7. 然后User1在分支 A 上做了提交commit4 
   ```mermaid
   gitGraph
       commit id: "commit1" tag: "Start of Branch A"
       branch A
       commit id: "commit2"
   		checkout main
       commit id: "commit5" tag: "Start of Branch B" type: highlight
       checkout A
       commit id: "commit3"
       checkout main
       commit id: "commit6" type: highlight
       checkout A
       commit id: "commit4"
   ```

   

8. 如果User1直接merge, commit按照生成的时间顺序排列

   ```mermaid
   gitGraph
       commit id: "commit1" tag: "Start of Branch A"
       commit id: "commit2"
       commit id: "commit5" tag: "Start of Branch B" type: highlight
       commit id: "commit3"
       commit id: "commit6" type: highlight
       commit id: "commit4"
       commit id: "resolve conflict merge commit"
   ```

   

9. 如果User1在merge之前先进行rebase
   ```mermaid
   gitGraph
       commit id: "commit1" tag: "Start of Branch A"
       commit id: "commit5" tag: "Start of Branch B" type: highlight
       commit id: "commit6" type: highlight
       branch A
       commit id: "commit2"
       commit id: "commit3"
       commit id: "commit4"
   ```

   ```mermaid
   gitGraph
       commit id: "commit1" tag: "Start of Branch A"
       commit id: "commit5" tag: "Start of Branch B" type: HIGHLIGHT
       commit id: "commit6" type: HIGHLIGHT
   
       commit id: "commit2"
       commit id: "commit3"
       commit id: "commit4"
   ```

   

7的状态下rebase的话, 以下是相应的 Git 命令：

```bash
$ git checkout main
$ git pull origin main
$ git checkout A
$ git pull origin A
$ git rebase main
# if conflicts exist, resolve them
# resolve commands 
# edit conflict file
$ vi <conflict file>
$ git add <conflict file>
$ git rebase --continue
# push branch A to remote repository (force push)
$ git push origin A -f
```



