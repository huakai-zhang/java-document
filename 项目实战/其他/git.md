Git 常用的是以下 6 个命令：**git clone**、**git push**、**git add** 、**git commit**、**git checkout**、**git pull**

![](https://note.youdao.com/yws/api/personal/file/WEBf942c817b5cce91c56a3a03bf3a4518d?method=getImage\&version=1258\&cstk=oo4z5tJt)

# git clone

```markdown
# cloen 并 clone 子模块
	git clone --recurse-submodules https://github.com/xxx/xxx.git
```

# branch

```markdown
# 查看全部本地分支
	git branch

# 查看全部远程分支
	git branch -r
```

# checkout

```markdown
# 拉取远程分支并创建本地分支
	git checkout -b test origin/test
```

# git pull

```markdown
git pull -rebase
```

# 提交流程

```sh
git add .
git commit -m '生产error问题修复'
git push
```

# 合并

```sh
git checkout B
git merge A
```

