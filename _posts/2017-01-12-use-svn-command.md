---
layout: post
title: 使用 svn 命令管理代码
date: 2017-01-12 16:16:16
tags: [linux,shell,svn]
categories:  
baseline:
---

### 检出项目代码
```
svn checkout $repository_url
```


### 列出有变化的文件
- 切换到项目代码根目录

```
cd /path/to/code_dir
svn status .
```


### 提交代码
- 切换到项目代码根目录

```
cd /path/to/code_dir
svn commit -m '修改记录' .
```


### 查看历史日志
- 切换到项目代码根目录

```
cd /path/to/code_dir
svn log -l 5 -v .
```


### 创建新分支
```
svn copy --parents -rHEAD -m ’创建新分支’ $trunk_repository_url $branches_repository_url
```


### 切换分支
- 切换到项目代码根目录

```
cd /path/to/code_dir
svn switch $need_switch_repository_url . -r HEAD --force
```


### 分支合并到主干
- 切换到项目代码根目录

```
cd /path/to/code_dir
```

- 将本地代码切换到主干

```
svn switch $trunk_repository_url . -r HEAD --force
```

- 更新分支代码到本地项目目录

```
svn merge -r123:HEAD $branches_repository_url .
```

- 提交代码到目的分支

```
svn commit -m '修改记录' .
```
