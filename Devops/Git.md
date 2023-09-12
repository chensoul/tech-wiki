## 克隆

### 只clone 最新一次提交

有时拉取大项目确实很耗时，只clone最近一次提交，会节省 clone时间。

```
git clone --depth=1 https://some.github.url/user/repo.git
```



## 分支

### 使用Git回溯到之前的commit

当你修改了一个配置文件，发现项目启动失败。当你重构了一些代码，发现程序出现bug。当你…。总是不管出现什么问题，你过去几个小时的努力可能要白费了，你想让一切恢复原状，那么就需要回溯到之前某一个正常的commit。

```bash
git reflog
f8fd780 (HEAD -> master, origin/master, origin/HEAD) HEAD@{0}: commit: 6/10
176bc3a HEAD@{1}: commit: xxxx
48b3b87 HEAD@{2}: commit: xxxx

git reset HEAD@{index}
```

第一个命令列出了过去提交的更改，可以看到第二列的格式`分支名称@{下标}`。第二个命令就可以帮我们的repo回到当时的状态。不管你中间执行了什么样的更改，git都可以把我们的项目带回当时的状态，神奇的时间机器。

## 撤消提交

有时候我们提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。 此时，可以运行带有 `--amend` 选项的提交命令来重新提交：

```bash
git add . # 或者 git add xxx.xx
git commit --amend
```

没有提交，但是希望修改提交信息：

 ```bash
 git commit --amend -m "new-message"
 ```

## 改动提交到了错误的分支

```bash
git reset HEAD~ --soft
git stash

git checkout CORRECT-branch
git stash pop
git add .
git commit -m "xxx"
```

经过上面的一串命令，改动就被移动到了正确的分支上。

还有另一种实现方式是使用cherry-pick。

```bash
git checkout CORRECT-branch
git cherry-pick master
git checkout master
git reset HEAD~ --hard
```



## 标签

### 打标签

```bash
git tag -a v1.2 -m "my version 1.2"
```

基于提交打标签：

```bash
git tag -a v1.2 9fceb02
```

### 推送标签

```bash
git push origin v1.2

#一次性推送很多标签
git push origin --tags
```

### 删除标签

```bash
#删除本地标签
git tag -d v1.2

#删除远程标签
git push origin --delete v1.2
```

### 从标签创建分支

```bash
git checkout -b version1.2 v1.2
```

