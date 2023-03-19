Git 常用的是以下 6 个命令：**git clone**、**git push**、**git add** 、**git commit**、**git checkout**、**git pull**

![](git.assets/git-command.jpg)

## git clone

```markdown
# cloen 并 clone 子模块
	git clone --recurse-submodules https://github.com/xxx/xxx.git
```

## branch

```markdown
# 查看全部本地分支
	git branch

# 查看全部远程分支
	git branch -r
```

## checkout

```markdown
# 拉取远程分支并创建本地分支
	git checkout -b test origin/test
```

## git pull

git pull -rebase

