# github协作



### 拉取PR到本地测试

```
git fetch origin pull/PRId/head:LocalBranchName
```

```
git fetch origin pull/1204/head:pr1204
```

```
git fetch upstream pull/PRId/head:LocalBranchName
```



### Rebase合并提交

```
git rebase -i HEAD~4
```

### commit

```
git commit --amend
1. 追加提交，它可以在不增加一个新的commit记录的情况下将新修改的代码追加到前一次的commit中但是请注意commit-id会改变
2. 覆盖上次提交的信息，也会生成一个新的commit-id
```



### PR

1. fork到自己的仓库
2. `git clone`到本地
3. `git remote add upstream [原项目地址]` 多添加一个源地址, 并命名为stream
4. `git status` `git add .` `git commit -m`代码commit
5. `git push origin master` 推到自己的GitHub的仓库
6. `git fetch upstream` 拉取原仓库
7. `git rebase upstream/master` 把原仓库修改的内容, 合并到当前分支
8. `git push origin master` 更新远程分支

```text
//先设置 upstream 为开源项目地址，目的是为了把开源项目的更新 同步到自己fork的项目中
git remote add upstream https://github.com/kubesphere/website.git
//获取更新
git fetch upstream
//合并更新
git rebase upstream/master
//如果有冲突呢 修改文件冲突 修改后git add 冲突文件名 commit提交
git rebase --continue
```

### 删除远程分支

```
git push origin --delete [branch]
```

