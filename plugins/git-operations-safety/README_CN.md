# Git Operations Safety 插件

## 插件作用

Git 操作安全规范插件,确保 Git 操作的安全性和可恢复性,防止数据丢失。在执行破坏性 Git 操作前提供安全检查和确认流程。

**核心价值:**
- 🛡️ 防止数据丢失 - 破坏性操作前强制确认
- 📋 安全检查清单 - 操作前后的完整检查流程
- 🚨 特殊保护 - 对 main/master 分支的 force push 特别限制
- 💾 备份策略 - 推荐使用 stash 和备份分支

---

## 运作方式

这是一个 **Skill (技能)**,在执行 Git 操作时自动激活,特别是破坏性操作。

---

## 破坏性操作 (需要确认)

以下操作会导致数据丢失或历史变更,**必须**获得明确的用户确认:

| 操作 | 风险 | 影响 |
|------|------|------|
| `git reset --hard` | 丢失所有未提交changes | ⚠️ 高风险 - 不可恢复 |
| `git push --force` | 覆盖远程历史 | ⚠️ 高风险 - 影响团队 |
| `git clean -fd` | 删除未跟踪文件 | ⚠️ 高风险 - 永久删除 |
| `git branch -D` | 强制删除分支 | ⚠️ 中等风险 - 丢失未合并工作 |

---

## 安全实践

### 1. 操作前检查

**规则**: 任何 Git 操作前总是运行 `git status`

```bash
# ✅ 标准工作流
git status                    # 1. 先检查状态
git add .                     # 2. 暂存变更
git status                    # 3. 再次确认
git commit -m "message"       # 4. 提交
```

### 2. reset --hard 的安全替代

**风险**: 丢失所有未提交的变更和暂存内容

**安全替代方案**:
```bash
# ✅ 使用 stash 保存变更
git stash push -m "临时保存"
git reset --hard HEAD

# 如需恢复:
git stash pop

# ✅ 创建备份分支
git branch backup-$(date +%Y%m%d-%H%M%S)
git reset --hard HEAD
```

### 3. force push 的安全规则

**风险**: 覆盖远程历史,影响其他团队成员

**安全替代**:
```bash
# ✅ 使用 --force-with-lease (更安全)
git push --force-with-lease
```

**特殊限制 - main/master 分支**:
```bash
# ❌❌❌ 绝对禁止 force push 到 main 分支
git push --force origin main
git push --force origin master

# 必须警告并要求二次确认
```

**警告模板**:
```
🚨 严重警告:

你正在尝试 force push 到 main 分支,这非常危险!

可能后果:
- 覆盖其他团队成员的提交
- 导致团队成员本地仓库冲突
- 可能丢失重要代码变更

这个操作通常不应该执行。

如果你真的需要这样做:
1. 通知所有团队成员
2. 确认没有人基于 main 分支工作
3. 确保存在完整备份

确定要继续吗?
```

### 4. clean 的安全实践

**风险**: 删除所有未跟踪文件和目录,不可恢复

**安全做法**:
```bash
# ✅ 先预览
git clean -fd --dry-run

# 确认后执行
git clean -fd
```

---

## 使用例子

### 例子 1: 安全的 Commit 工作流

```bash
# 1. 检查状态
git status

# 2. 查看changes
git diff

# 3. 添加文件
git add src/auth.ts src/user.ts

# 4. 再次确认
git status

# 5. 提交
git commit -m "feat(auth): 添加用户认证"

# 6. 推送前确认
git status

# 7. 推送
git push origin feature-auth
```

### 例子 2: 处理危险操作请求

**场景**: 用户请求 "git reset --hard"

**✅ 正确处理**:
1. 运行 `git status`
2. 显示将要丢失的内容:
   ```
   ⚠️ 警告: 将丢失以下未提交的变更:
   - src/auth.ts
   - src/user.ts

   继续?
   ```
3. 等待用户确认
4. 建议先备份
5. 用户确认后执行

---

## 安全检查清单

### 提交前

- [ ] 运行 `git status`
- [ ] 运行 `git diff` 查看变更
- [ ] 确认所有变更都是有意的
- [ ] 确认无敏感信息 (密码、密钥)
- [ ] 确认无不应提交的文件
- [ ] 提交消息遵循规范

### 推送前

- [ ] 通过所有本地质量检查
- [ ] 确认推送到正确的分支
- [ ] 如果 force push,确认影响范围
- [ ] 不推送到 main/master (除非有权限)
- [ ] 已通知团队成员 (如果是共享分支)

### 破坏性操作前

- [ ] 运行 `git status` 了解当前状态
- [ ] 创建备份分支或使用 stash
- [ ] 理解操作的确切影响
- [ ] 获得明确的用户确认
- [ ] 准备好回滚计划

---

## 恢复程序

### 恢复意外删除的提交

```bash
# 查看操作历史
git reflog

# 恢复到删除前
git reset --hard HEAD@{1}
```

### 恢复被 force push 覆盖的提交

```bash
# 在受影响成员的机器上
git reflog

# 创建分支保存被覆盖的提交
git branch recovery <commit-hash>
```

---

## 推荐工具

### Git Hooks

```bash
# pre-push hook: 阻止直接推送到 main
#!/bin/sh
protected_branch='main'
current_branch=$(git symbolic-ref HEAD | sed -e 's,.*/\(.*\),\1,')

if [ $current_branch = $protected_branch ]; then
    echo "⚠️  不允许直接推送到 main!"
    exit 1
fi
```

### Git Aliases

```bash
# ~/.gitconfig
[alias]
    # 安全的 force push
    pushf = push --force-with-lease

    # 提交前预览
    preview = diff --staged

    # 安全的 clean
    cleancheck = clean -fd --dry-run
```

---

## 常见问题

**Q: 为什么每次操作前都要运行 git status?**
A: 它显示当前状态、修改的文件、暂存的内容、未跟踪的文件、与远程分支的关系,帮助你了解操作的影响。

**Q: --force-with-lease 和 --force 有什么区别?**
A: `--force-with-lease` 会检查远程分支是否被其他人更新,如果是则拒绝推送,更安全。

**Q: 什么情况下可以 force push 到 main?**
A: 几乎没有。只有在紧急情况下(如误提交敏感信息)并获得团队批准后才可以。

---

## 相关插件

- **git-commit-conventions** - Git 提交规范
- **pull-request-guidelines** - PR 创建规范

---

## 作者

- 作者: LeekJay
- 邮箱: leekjay@foxmail.com
- 仓库: https://github.com/LeekJay/claude-skills-plugin
