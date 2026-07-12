# Git 原理与常用操作学习笔记

> 本文整理了从“本地新建项目并同步到 GitHub”这一完整实践过程中，需要掌握的 Git 核心概念、内部机理和常用命令。

---

## 1. Git 到底是什么

Git 是一个**分布式版本控制系统**。

## feature-a branch test
## 冲突B
## 冲突a

它的核心作用不是简单地“保存文件”，而是：

- 记录项目在不同时间点的状态；
- 保存文件的修改历史；
- 支持多人协作；
- 支持创建分支并行开发；
- 支持回退、比较和合并不同版本；
- 可以在本地独立工作，不依赖网络。

GitHub 则是一个托管 Git 仓库的网站。

可以这样区分：

```text
Git      = 本地版本控制工具
GitHub   = 远程 Git 仓库托管平台
```

即使没有 GitHub，也可以在本地使用 Git：

```bash
git init
git add .
git commit -m "Save current version"
git log
```

只有执行 `git push`、`git pull`、`git fetch` 等远程操作时，才需要连接 GitHub。

---

## 2. Git 最重要的整体模型

学习 Git 时，最重要的是理解以下四个区域：

```text
工作区 Working Tree
        │
        │ git add
        ▼
暂存区 Staging Area / Index
        │
        │ git commit
        ▼
本地仓库 Local Repository
        │
        │ git push
        ▼
远程仓库 Remote Repository
```

### 2.1 工作区

工作区就是平时直接看到和修改的项目文件：

```text
my-project/
├── README.md
├── app.py
└── config.yaml
```

在 VS Code 中编辑文件，就是在修改工作区。

文件在工作区中可能处于以下状态：

- `untracked`：Git 尚未跟踪的新文件；
- `modified`：已跟踪文件发生了修改；
- `unmodified`：文件与当前版本一致。

---

### 2.2 暂存区

暂存区又叫：

```text
Staging Area
Index
```

它本质上不是一个普通文件夹，而是 Git 内部维护的一份索引。

它描述的是：

> 下一次提交准备包含什么内容。

执行：

```bash
git add README.md
```

并不是上传到 GitHub，也不是创建正式版本。

它真正做的是：

1. 读取 `README.md` 当前内容；
2. 将文件内容写入 Git 对象数据库；
3. 更新暂存区；
4. 指定下一次提交使用这个版本的文件。

因此，可以把暂存区理解为：

```text
下一次提交的设计稿
```

---

### 2.3 本地仓库

本地仓库主要位于项目目录中的：

```text
.git/
```

执行：

```bash
git commit
```

后，Git 会根据暂存区创建一个正式的本地历史节点。

这个提交只存在于本地电脑中，不会自动上传到 GitHub。

即使断网，也可以执行：

```bash
git add
git commit
git log
git branch
git switch
```

---

### 2.4 远程仓库

远程仓库是其他位置上的 Git 仓库，例如：

- GitHub；
- GitLab；
- 公司内部 Git 服务器；
- 另一台 Linux 服务器。

常见远程仓库别名是：

```text
origin
```

例如：

```bash
git remote add origin git@github.com:username/project.git
```

这里的 `origin` 只是该远程地址在本地的简称。

---

## 3. Git 的核心不是“差异”，而是快照和对象

从使用者角度，Git 经常显示文件之间的差异：

```diff
- Hello
+ Hello Git
```

但从内部数据结构看，Git 更接近于：

> 每次提交都指向一份完整项目快照。

Git 主要有四类对象：

```text
blob    文件内容
tree    目录结构
commit  提交节点
tag     固定标签
```

其中最重要的是：

```text
blob
tree
commit
```

---

### 3.1 Blob 对象

Blob 用于保存文件内容。

例如：

```text
README.md 内容：Hello Git
```

Git 会根据文件内容计算哈希，并保存为一个 blob 对象。

概念上：

```text
blob
├── 哈希值
└── 文件内容
```

Blob 通常不保存原始文件名。

文件名与 blob 的对应关系由 tree 对象保存。

如果两个文件内容完全相同，它们可以共享同一个 blob 对象。

---

### 3.2 Tree 对象

Tree 对象用于保存目录结构。

例如：

```text
tree
├── README.md → blob A
├── app.py    → blob B
└── src/      → tree C
```

Tree 记录：

- 文件名；
- 文件模式；
- 文件对应的 blob；
- 子目录对应的 tree。

项目根目录对应的根 tree 可以描述整个项目快照。

---

### 3.3 Commit 对象

Commit 对象描述一次正式提交。

它通常包含：

```text
commit
├── 根 tree
├── 父 commit
├── 作者
├── 提交者
├── 时间
└── 提交说明
```

多个提交通过父提交关系连接：

```text
A ← B ← C ← D
```

含义是：

```text
D 的父提交是 C
C 的父提交是 B
B 的父提交是 A
```

因此 Git 历史本质上是一个由提交节点组成的有向无环图。

---

## 4. `git init`：初始化本地仓库

命令：

```bash
git init
```

作用：

> 在当前目录中创建一套 Git 仓库所需的内部结构。

执行后通常会生成：

```text
.git/
```

---

### 4.1 `git init` 不会做什么

`git init` 不会：

- 上传任何文件；
- 创建 GitHub 仓库；
- 自动提交现有文件；
- 自动跟踪所有文件；
- 修改项目文件；
- 自动添加远程仓库。

例如项目中已有：

```text
README.md
app.py
```

执行：

```bash
git init
```

后，这些文件仍然只是未跟踪文件。

查看：

```bash
git status
```

可能看到：

```text
Untracked files:
  README.md
  app.py
```

---

### 4.2 `.git` 目录内部结构

典型结构：

```text
.git/
├── HEAD
├── config
├── index
├── objects/
├── refs/
├── hooks/
└── logs/
```

#### `.git/objects`

保存 Git 对象：

```text
blob
tree
commit
tag
```

#### `.git/index`

保存暂存区信息。

`git add` 主要会更新这里。

#### `.git/HEAD`

表示当前所在的分支或提交位置。

例如：

```text
ref: refs/heads/main
```

意思是：

```text
HEAD 当前指向 main 分支
```

#### `.git/refs/heads/main`

保存本地 `main` 分支当前指向的 commit 哈希。

概念上：

```text
main → commit D
```

#### `.git/config`

保存当前仓库配置，例如：

```ini
[remote "origin"]
    url = git@github.com:username/project.git
```

也会保存分支跟踪关系。

---

### 4.3 删除 `.git` 会发生什么

删除 `.git` 后，项目文件通常仍然存在，但 Git 信息会丢失：

- 提交历史；
- 分支；
- 暂存区；
- 远程仓库配置；
- 本地尚未推送的提交。

因此 `.git` 是整个仓库的核心。

---

## 5. `git add`：准备下一次提交

常见命令：

```bash
git add README.md
git add .
git add -A
```

最准确的理解是：

> 把指定路径当前的状态写入暂存区，准备进入下一次提交。

---

### 5.1 新文件第一次 add

假设：

```text
README.md = Version 1
```

执行前：

```text
工作区：Version 1
暂存区：没有 README.md
HEAD：没有 README.md
```

执行：

```bash
git add README.md
```

之后：

```text
工作区：Version 1
暂存区：Version 1
HEAD：没有 README.md
```

---

### 5.2 add 后继续修改文件

这是理解暂存区最重要的例子。

先写：

```text
Version 1
```

然后：

```bash
git add README.md
```

之后又把工作区修改为：

```text
Version 2
```

此时：

```text
工作区：Version 2
暂存区：Version 1
HEAD：尚无该文件
```

如果立即执行：

```bash
git commit -m "Add README"
```

提交进去的是：

```text
Version 1
```

不是 Version 2。

因为：

> `git commit` 默认读取暂存区，而不是直接读取工作区。

要提交 Version 2，需要再次执行：

```bash
git add README.md
```

---

### 5.3 `git add` 不只是添加新文件

它还可以暂存：

- 新增；
- 修改；
- 删除；
- 重命名。

因此更准确地说：

```text
git add = 把路径当前状态同步到暂存区
```

---

### 5.4 `git add .` 和 `git add -A`

```bash
git add .
```

表示处理当前目录及其子目录中的变化。

它的作用范围与当前所在目录有关。

```bash
git add -A
```

通常表示处理整个仓库中的所有变化：

- 新文件；
- 修改；
- 删除。

对于初学阶段：

```bash
git status
git add .
git status
```

是比较直观的工作方式。

---

### 5.5 `.gitignore`

`.gitignore` 用于忽略不应该被 Git 跟踪的文件。

例如：

```gitignore
*.log
.env
*.key
*.pem
node_modules/
.vscode/
.idea/
```

但 `.gitignore` 主要对尚未被跟踪的文件生效。

如果 `.env` 已经被提交，再写入 `.gitignore` 也不会自动停止跟踪。

可以使用：

```bash
git rm --cached .env
```

停止跟踪但保留本地文件。

---

### 5.6 查看不同区域的差异

查看工作区与暂存区之间的差异：

```bash
git diff
```

回答：

> 哪些修改还没有执行 `git add`？

查看暂存区与当前提交之间的差异：

```bash
git diff --staged
```

或：

```bash
git diff --cached
```

回答：

> 下一次 commit 会提交什么？

查看工作区整体与当前提交之间的差异：

```bash
git diff HEAD
```

---

## 6. `git commit`：创建本地历史节点

命令：

```bash
git commit -m "Add README"
```

作用：

> 根据暂存区创建项目快照和 commit 对象，并移动当前分支指针。

---

### 6.1 Commit 保存什么

一个 commit 通常保存：

```text
根 tree
父 commit
作者 author
提交者 committer
作者时间
提交时间
提交说明
```

例如：

```text
commit abc123
tree def456
parent 789xyz
author Arthur
committer Arthur

Add README
```

---

### 6.2 Commit 不是上传

执行：

```bash
git commit
```

只是保存在本地。

它不会自动：

- 上传到 GitHub；
- 通知其他开发者；
- 更新远程仓库。

上传需要执行：

```bash
git push
```

---

### 6.3 分支本质上是指针

提交对象本身通常不知道自己属于哪个分支。

分支只是一个可以移动的指针。

例如：

```text
A ← B ← C
        ↑
       main
```

创建新提交 D 后：

```text
A ← B ← C ← D
            ↑
           main
```

发生了两件事：

1. 创建 Commit D；
2. 将 `main` 从 C 移动到 D。

---

### 6.4 HEAD 是什么

HEAD 表示当前工作位置。

正常情况下：

```text
HEAD → main → Commit D
```

也就是说：

- HEAD 指向 `main`；
- `main` 指向 Commit D。

执行新的 commit 后：

```text
HEAD → main → Commit E
```

---

### 6.5 Git 保存的是快照，但不会重复复制所有文件

逻辑上，每个提交都对应完整项目快照：

```text
Commit A → Snapshot A
Commit B → Snapshot B
Commit C → Snapshot C
```

但如果文件没有变化，多个快照可以共享相同 blob：

```text
Commit A:
  README.md → blob 1
  app.py    → blob 2

Commit B:
  README.md → blob 3
  app.py    → blob 2
```

`app.py` 没有变化，因此继续复用 blob 2。

这实现了：

```text
逻辑上：完整快照
存储上：复用未变化对象
```

---

### 6.6 Commit 默认只提交暂存区

假设：

```text
README.md 已暂存
app.py 已修改但未暂存
```

执行：

```bash
git commit -m "Update README"
```

只会提交 README 的修改。

这也是暂存区的重要价值：

> 可以把混杂的修改拆分成多个逻辑清晰的 commit。

---

### 6.7 `git commit -am`

```bash
git commit -am "Fix bug"
```

其中：

```text
-a = 自动暂存已跟踪文件的修改和删除
-m = 指定提交说明
```

但它不会自动加入全新的未跟踪文件。

因此新文件仍然需要：

```bash
git add new-file.txt
```

初学阶段建议优先使用：

```bash
git add ...
git commit -m "..."
```

这样每一步更清楚。

---

## 7. `git push`：把本地对象和分支更新发送到远程

常见命令：

```bash
git push -u origin main
```

核心含义：

> 把远程缺少的 Git 对象发送过去，并请求远程将 main 分支更新到本地 main 当前指向的 commit。

---

### 7.1 Push 不是只上传当前文件

假设本地：

```text
A ← B ← C
        ↑
       main
```

远程：

```text
A
↑
远程 main
```

执行：

```bash
git push origin main
```

Git 通常会发送：

- Commit B；
- Commit C；
- 它们依赖的 tree；
- 它们依赖的 blob。

然后请求远程：

```text
main：A → C
```

---

### 7.2 `origin` 和 `main`

```bash
git push origin main
```

含义：

```text
origin = 目标远程仓库
main   = 要推送的本地分支
```

查看远程地址：

```bash
git remote -v
```

例如：

```text
origin  git@github.com:username/project.git
```

---

### 7.3 `-u` 是什么

```bash
git push -u origin main
```

其中：

```text
-u = --set-upstream
```

作用：

> 建立本地 main 与远程 origin/main 的跟踪关系。

建立后：

```text
本地 main ↔ origin/main
```

以后可以直接执行：

```bash
git push
git pull
```

不需要每次写：

```bash
git push origin main
git pull origin main
```

通常一个新分支第一次推送时使用一次 `-u` 即可。

查看跟踪关系：

```bash
git branch -vv
```

---

### 7.4 Fast-forward

假设远程：

```text
A ← B
    ↑
远程 main
```

本地：

```text
A ← B ← C ← D
            ↑
本地 main
```

将远程 main 从 B 移动到 D，不会丢失任何历史，因为 B 是 D 的祖先。

这叫：

```text
fast-forward
```

---

### 7.5 Non-fast-forward

远程：

```text
A ← B ← X
```

本地：

```text
A ← B ← C
```

如果直接把远程 main 从 X 改到 C，X 会从 main 历史中消失。

因此普通 push 会被拒绝：

```text
non-fast-forward
```

通常要先整合远程更新：

```bash
git pull --rebase
```

或：

```bash
git fetch
git merge origin/main
```

然后再：

```bash
git push
```

---

## 8. SSH 连接 GitHub

GitHub 可以通过 HTTPS 或 SSH 连接。

SSH 适合长期使用，因为配置完成后通常不需要重复登录。

---

### 8.1 生成 SSH Key

推荐使用 Ed25519：

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

默认会生成：

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

其中：

```text
id_ed25519       私钥
id_ed25519.pub   公钥
```

私钥绝对不能：

- 上传到 GitHub；
- 放进项目目录；
- 提交到 Git；
- 发给其他人。

---

### 8.2 启动 ssh-agent

Git Bash 中：

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

检查：

```bash
ssh-add -l
```

---

### 8.3 复制公钥

Windows Git Bash：

```bash
clip < ~/.ssh/id_ed25519.pub
```

然后将公钥添加到：

```text
GitHub
→ Settings
→ SSH and GPG keys
→ New SSH key
```

---

### 8.4 测试连接

```bash
ssh -T git@github.com
```

成功时可能看到：

```text
Hi username! You've successfully authenticated,
but GitHub does not provide shell access.
```

其中：

```text
GitHub does not provide shell access
```

不是报错，只表示 GitHub 允许通过 SSH 使用 Git，但不提供普通服务器 Shell。

---

### 8.5 SSH 仓库地址

格式：

```text
git@github.com:用户名/仓库名.git
```

例如：

```bash
git remote add origin git@github.com:ArthurLu/git-learning.git
```

注意：

```text
git@github.com
```

中的 `git` 是 GitHub SSH 使用的固定用户，不是自己的 GitHub 用户名。

---

### 8.6 HTTPS 地址改成 SSH

如果已经存在 origin：

```bash
git remote -v
```

可以修改：

```bash
git remote set-url origin git@github.com:username/project.git
```

不要再次执行：

```bash
git remote add origin ...
```

否则可能出现：

```text
remote origin already exists
```

---

### 8.7 工作网络封锁 SSH 22 端口

可以测试 GitHub SSH 的 443 端口：

```bash
ssh -T -p 443 git@ssh.github.com
```

如果可用，可以配置：

```text
~/.ssh/config
```

内容：

```sshconfig
Host github.com
    HostName ssh.github.com
    User git
    Port 443
```

之后仍然可以使用：

```bash
ssh -T git@github.com
```

---

## 9. 从本地新项目首次推送到 GitHub

假设本地已有项目目录和 `README.md`。

### 9.1 配置身份

```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

查看：

```bash
git config --global --list
```

---

### 9.2 初始化仓库

```bash
git init
```

查看状态：

```bash
git status
```

---

### 9.3 暂存文件

```bash
git add README.md
```

或：

```bash
git add .
```

---

### 9.4 创建第一次提交

```bash
git commit -m "Add initial README"
```

---

### 9.5 将主分支命名为 main

```bash
git branch -M main
```

查看：

```bash
git branch
```

---

### 9.6 在 GitHub 创建空仓库

由于本地已经存在第一次提交，GitHub 创建仓库时不要勾选：

```text
Add a README
Add .gitignore
Choose a license
```

否则本地和远程可能分别产生不同的初始提交。

---

### 9.7 添加 SSH 远程地址

```bash
git remote add origin git@github.com:username/repository.git
```

检查：

```bash
git remote -v
```

---

### 9.8 第一次推送

```bash
git push -u origin main
```

以后：

```bash
git push
```

---

## 10. 日常最常用工作流

修改文件后：

```bash
git status
git diff
git add .
git diff --staged
git commit -m "Describe the change"
git push
```

简化理解：

```text
修改文件
   ↓
git add
   ↓
准备下一次提交
   ↓
git commit
   ↓
保存到本地历史
   ↓
git push
   ↓
同步到远程仓库
```

---

## 11. 其他常用命令

### 11.1 `git status`

```bash
git status
```

用于查看：

- 当前分支；
- 未跟踪文件；
- 未暂存修改；
- 已暂存修改；
- 与远程分支的领先或落后情况。

这是最安全、最应该频繁执行的命令。

---

### 11.2 `git log`

```bash
git log
```

简洁查看：

```bash
git log --oneline
```

查看图形化分支历史：

```bash
git log --oneline --graph --decorate --all
```

---

### 11.3 `git branch`

查看本地分支：

```bash
git branch
```

创建分支：

```bash
git branch feature
```

分支本质是一个指向 commit 的可移动指针。

---

### 11.4 `git switch`

切换分支：

```bash
git switch feature
```

创建并切换：

```bash
git switch -c feature
```

---

### 11.5 `git fetch`

```bash
git fetch origin
```

作用：

- 下载远程新对象；
- 更新本地的 `origin/main` 等远程跟踪引用；
- 不自动修改当前工作区；
- 不自动合并到当前分支。

因此 `git fetch` 相对安全。

---

### 11.6 `git pull`

```bash
git pull
```

可以粗略理解为：

```text
git fetch
+
git merge
```

如果配置了 rebase，也可能是：

```text
git fetch
+
git rebase
```

因此 pull 不只是下载，还会整合远程历史。

---

### 11.7 `git clone`

```bash
git clone git@github.com:username/project.git
```

它大致会完成：

1. 创建目录；
2. 初始化本地仓库；
3. 添加 origin；
4. 下载对象；
5. 创建远程跟踪分支；
6. 创建本地默认分支；
7. 检出工作区；
8. 设置 upstream。

---

### 11.8 `git remote`

查看远程：

```bash
git remote -v
```

添加远程：

```bash
git remote add origin <URL>
```

修改地址：

```bash
git remote set-url origin <URL>
```

---

## 12. `main`、`origin/main` 和远程 main 的区别

这是非常容易混淆的地方。

```text
main
```

是本地工作分支。

```text
origin/main
```

是本地保存的“上一次知道的远程 main 状态”。

真正的远程 `main` 位于 GitHub 服务器上。

因此实际有三者：

```text
本地 main
本地 origin/main
GitHub 上真实的 main
```

执行：

```bash
git fetch origin
```

后，本地 `origin/main` 才会更新。

---

## 13. 常见错误

### 13.1 `Author identity unknown`

说明未配置用户名和邮箱：

```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

---

### 13.2 `remote origin already exists`

说明已经有名为 origin 的远程仓库。

查看：

```bash
git remote -v
```

修改：

```bash
git remote set-url origin <correct-url>
```

---

### 13.3 `src refspec main does not match any`

常见原因：

1. 尚未创建任何 commit；
2. 本地分支不叫 main。

解决：

```bash
git add .
git commit -m "Initial commit"
git branch -M main
git push -u origin main
```

---

### 13.4 `repository not found`

常见原因：

- 用户名错误；
- 仓库名错误；
- 远程 URL 错误；
- 当前 GitHub 账号无权限；
- 私有仓库没有访问权限。

查看：

```bash
git remote -v
```

---

### 13.5 `Permission denied (publickey)`

SSH 认证失败。

检查：

```bash
ssh -T git@github.com
ssh-add -l
```

可能需要：

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

并确认公钥已添加到正确的 GitHub 账号。

---

### 13.6 `non-fast-forward`

说明远程存在本地没有的提交。

通常先：

```bash
git pull --rebase
```

解决冲突后：

```bash
git push
```

不要在不了解影响的情况下随意使用：

```bash
git push --force
```

因为可能覆盖远程历史。

---

## 14. 工作电脑上的安全注意事项

向个人 GitHub 上传前，需要确认公司政策。

不要提交：

```text
客户日志
客户 IP
账号密码
API Key
Access Token
私钥
证书
内部文档
未公开代码
生产配置
.env 文件
```

尤其注意：

```text
即使之后从当前目录删除，
敏感内容仍可能保留在 Git 历史中。
```

建议在第一次 commit 前就配置 `.gitignore`。

---

## 15. 最重要的心智模型

不要只把 Git 记成一串命令：

```text
init → add → commit → push
```

应该理解为：

```text
git init
创建仓库基础结构、对象数据库、索引和引用

git add
将工作区指定内容写入对象数据库，并更新暂存区

git commit
根据暂存区创建 tree 和 commit，并移动当前分支指针

git push
把远程缺少的对象发送过去，并更新远程分支引用
```

最终可以浓缩为：

```text
工作区是正在编辑的内容；
暂存区是下一次提交的设计稿；
commit 是不可变的历史快照节点；
分支是指向 commit 的可移动指针；
HEAD 表示当前工作位置；
push 是同步对象并更新远程引用。
```

---

## 16. 推荐复习命令

```bash
git status
git diff
git diff --staged
git add .
git commit -m "message"
git log --oneline --graph --decorate --all
git branch -vv
git remote -v
git fetch
git pull
git push
```

其中最值得形成习惯的是：

```bash
git status
```

在执行重要操作前后，都可以先看一次当前状态。

---

## 17. 一套标准实践模板

### 新建项目并首次推送

```bash
cd project-directory

git init
git status

git add .
git diff --staged

git commit -m "Initial commit"

git branch -M main

git remote add origin git@github.com:username/repository.git
git remote -v

git push -u origin main
```

### 后续日常修改

```bash
git status
git diff

git add .

git diff --staged
git commit -m "Describe the change"

git push
```

### 开始工作前同步远程

```bash
git pull
```

如果想先安全查看远程变化：

```bash
git fetch
git log --oneline --graph --decorate --all
```

---

# 结语

真正理解 Git 的关键，不是死记命令，而是始终判断：

1. 当前修改位于工作区、暂存区还是提交历史？
2. HEAD 和当前分支指向哪个 commit？
3. 本地 main 与 origin/main 是否一致？
4. 当前命令是在创建对象，移动引用，还是同步远程？

当这些问题能够回答清楚时，绝大多数 Git 命令都会变得容易理解。