# 🌿 分支与合并冲突实战练习笔记

> 本文记录一次完整的"分支 → Fast-forward → 制造冲突 → 解决冲突"的动手练习过程，以及中途踩到的坑。

---

## 🎯 今日练习目标

1. 掌握 `git switch -c` 创建并切换分支
2. 亲眼看到 **Fast-forward 合并** 长什么样
3. 主动制造一次**合并冲突**并手动解决
4. 学会从"未完成的 merge"中脱身（`MERGE_HEAD` 报错）

---

## 1. 第一次尝试：Fast-forward 合并（没有冲突）

### 操作步骤

```bash
git switch -c feature-a          # 创建并切换到 feature-a
# 修改 README.md
git commit -am "feature-a branch test"

git switch main                  # 回到 main
git merge feature-a              # 合并
```

### 结果

```text
Updating 3abe90f..bb0ec85
Fast-forward
 README.md                        | 2 ++
 homePC_test_file_from_another_PC.txt | 1 +
 2 files changed, 3 insertions(+)
```

### 💡 为什么没冲突？

**Fast-forward（快进）的必要条件**：`main` 从分岔点之后**没有任何新提交**。

```text
main:      A ── B                    ← main 停在 B 没动
                 \
feature-a:        C ── D             ← 新提交都在这边
```

Git 只需要把 `main` 指针"往前挪"到 `D`，不用真正做合并 → **不可能产生冲突**。

---

## 2. 第二次尝试：主动制造冲突

### 关键：让两个分支都动

```bash
git switch -c feature-a
# 修改 README.md 第 X 行为 "feature-a 的版本"
git commit -am "feature-a 冲突"

git switch main
# 修改 README.md 同一行为 "main 的版本"   ← 这一步是关键
git commit -am "冲突B"

git merge feature-a              # ← 这次真的冲突了
```

### 分支图

```text
* 893f5a2 冲突B          ← 合并 commit（解决冲突后生成）
|\
| * b37aaa7 feature-a 冲突
* | 8d964bb 冲突B
|/
* bb0ec85 feature-a branch test
```

两条线各走各的，Git 不知道保留哪个版本，所以让人工介入。

---

## 3. 中途踩坑：`MERGE_HEAD exists` 报错

### 错误信息

```text
fatal: You have not concluded your merge (MERGE_HEAD exists).
Please, commit your changes before you merge.
```

### 原因

上一次 merge 触发冲突后，**既没解决完提交、也没放弃**，Git 就把仓库锁在"合并进行中"的状态。`MERGE_HEAD` 这个文件的存在就是这个状态的标志。

### 三种出路

| 场景 | 命令 |
|---|---|
| 想放弃这次 merge，回到干净状态 | `git merge --abort` |
| 想解决冲突继续 merge | 改文件 → `git add` → `git commit` |
| 想彻底重来 | `git merge --abort` + `git reset --hard HEAD` |

### 通用规律

以后看到这类提示都是同一个意思 —— **有个操作没走完，先了结它**：

- `(MERGE_HEAD exists)` → `git merge --abort` 或完成 merge
- `(rebase in progress)` → `git rebase --abort` 或 `git rebase --continue`
- `(cherry-picking)` → `git cherry-pick --abort` 或 `--continue`

---

## 4. 冲突解决完整流程

```bash
# 1. 查看哪些文件冲突
git status                       # 找 "Unmerged paths"

# 2. 打开冲突文件，会看到这样的标记：
#    <<<<<<< HEAD
#    main 分支的内容
#    =======
#    feature-a 分支的内容
#    >>>>>>> feature-a
#
#    → 手动删掉标记，留下最终想要的内容

# 3. 标记为已解决
git add <冲突的文件>

# 4. 完成合并（会自动生成 merge commit）
git commit
```

---

## 5. 额外知识点

### 5.1 `--no-ff`：强制生成合并 commit

即使可以 fast-forward，也可以强制留下一个 merge commit：

```bash
git merge --no-ff feature-a
```

**用途**：团队协作时能在历史图上清楚看到"这里合并了一个功能分支"。

### 5.2 `git push` 不能带 message

`push` 只负责搬运已有的 commit，message 是 commit 的属性：

```bash
git commit -m "xxx"     # message 在这里定
git push                # push 不接受 -m
```

如果想改 message 再推：

```bash
git commit --amend -m "新 message"
git push                # 未推送过的 commit 直接推
# 已推送过的需要 git push --force（有风险）
```

如果想给远程留个"标记"，用 **tag**：

```bash
git tag -a v1.0 -m "第一个稳定版本"
git push origin v1.0
```

### 5.3 `git switch` 找不到仓库的报错

```text
fatal: not a git repository (or any of the parent directories): .git
```

**原因**：当前目录及其**父目录**都没有 `.git` 文件夹。Git 只往上找，不往下找子目录。

**排查**：`ls -la` 看当前目录，或 `pwd` 确认路径，先 `cd` 到含 `.git` 的目录再执行。

---

## 6. 今日心智模型总结

1. **Fast-forward vs 真正合并**：只有两边都有新提交时，才需要真正的合并（也才可能冲突）。
2. **冲突不可怕**：本质是 Git 让人工挑选保留哪个版本。
3. **中途卡住有救**：几乎所有"卡在半路"的状态都有 `--abort` 可以放弃。
4. **push 只是搬运工**：所有"内容层面"的信息（message、作者）都属于 commit。

---

## 7. 下一步练习计划

- [ ] 撤销操作：`restore` / `reset` / `revert` / `reflog`（救命符）
- [ ] 交互式 rebase：`git rebase -i` 的 squash / reword / fixup
- [ ] `--amend` 改上一次 commit
- [ ] `git stash` 临时保存工作区
