# Git & GitHub CLI 系列路线图

> 内部规划文档，不发布。
> 定位：命令行系列之后的第二个长系列。单人使用为主 + 开源协作。
> AI 时代背景：AI 降低了理解代码库和写 PR 的门槛，Git/GitHub 操作本身成为新的瓶颈。

---

## Part 1：基础篇——上手与日常操作

### 1. 📖 版本控制为什么存在

为什么需要 Git？从"手动备份"的痛点说起。

### 2. 🔧 `git init` / `git clone` / `.gitignore`

初始化仓库、克隆已有仓库、忽略文件的配置。

### 3. 📝 `git add` 深入——暂存区到底是什么

`add` 不只是"添加文件"——暂存区（index）的概念、为什么分 staged/unstaged。

### 4. 📝 `git commit` 深入——commit、tree、blob 的关系

commit 背后是什么结构？hash 怎么来的？commit message 怎么写——AI 帮忙写但人要会判断好坏。

### 5. 📋 `git status` / `git log`——看懂仓库的状态和历史

`status` 的各种状态含义，`log` 的格式化、筛选、图形化输出。

---

## Part 2：分支与协作篇

### 6. 🌿 `git branch` 深入——分支的本质只是一个指针

`HEAD`、`branch`、`tag` 的本质。创建、切换、删除分支。

### 7. 🔀 `git merge`——合并的三种类型

fast-forward、recursive、octopus merge。`git merge --no-ff` 什么时候用。

### 8. 🕯️ 冲突深度拆解

冲突的产生、冲突标记格式、手动解决、`git mergetool`。结合 AI 辅助解决冲突。

### 9. 🔄 `git rebase`——rebase 比 merge 好在哪

rebase vs merge 的对比。交互式 rebase 的基础。什么时候不该 rebase。

---

## Part 3：历史操作篇

### 10. 🕵️ `git reset` / `git revert` / `git restore`——三条撤销之路

三条路径的对比：reset（移动 HEAD）、revert（反向提交）、restore（恢复文件）。--soft / --mixed / --hard 的区别。

### 11. 📦 `git stash`——临时保存

`stash push`、`pop`、`apply`、`drop`、`list`。stash 和分支切换的关系。

### 12. ✏️ `git commit --amend` / `git rebase -i`——重写历史

`amend` 修改最近一次 commit。`rebase -i` 修改、合并、删除历史 commit。

---

## Part 4：远程与开源篇

### 13. 🌐 `git remote` / `git push` / `git pull` / `git fetch`——远程完整图景

remote 的增删改查。push/pull/fetch 的区别。上游分支与 tracking。SSH key 配置、`gh auth`。

### 14. 🎫 `gh` 入门——GitHub CLI 的基本操作

`gh issue`、`gh pr`、`gh repo`、`gh run`。为什么用 gh 而不是网页。

### 15. 🔄 PR 工作流实战

从 fork、clone、branch、push 到提 PR。`gh pr create`、`gh pr checkout`。单人开源贡献的完整流程。

### 16. ✅ 用 `gh` + AI 做开源贡献

AI 时代做开源的门槛变化：用 AI 理解陌生代码库、写 PR 描述、处理 review feedback。`gh pr review`、`gh pr checks`。

---

## Part 5：深入与收尾篇

### 17. ⚙️ `git diff` / `git blame` / `git bisect`——调试与追溯

`diff` 的多种比较方式。`blame` 追溯代码来源。`bisect` 二分查找找 bug。

### 18. 🧠 Git 内部探秘——`.git` 目录里有什么（可选深度）

`.git` 目录结构、objects、refs、HEAD。理解 Git 的数据模型。

---

## 体量

**共 18 篇**——与 Command Line 系列规模接近。

---

## 定位说明

- 不是"Git 从入门到放弃"——是**面向已经懂终端的读者**的深入 Git 系列
- 假设读者跑完了命令行系列，能熟练使用终端
- 每个命令独立成篇，讲透原理 + 实际场景
- AI 时代做开源的门槛变化——从"不敢碰开源代码"到"能用 gh + AI 参与"是系列的一条暗线

---

*初稿框架 · 2026-07-14*
