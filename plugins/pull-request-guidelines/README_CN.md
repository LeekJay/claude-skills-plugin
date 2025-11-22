# Pull Request Guidelines 插件

## 插件作用

标准化 Pull Request 工作流和内容要求,确保 PR 包含完整信息、通过质量检查,便于代码审查和追溯。

**核心价值:**
- 📝 标准化 PR 格式 - 统一的 PR 标题和描述规范
- ✅ 完整验证步骤 - 自动化和手动测试清单
- 🔍 影响评估 - 明确 PR 的影响范围和风险
- 🇨🇳 中文总结 - 清晰准确的变更说明

---

## PR 标题格式

遵循 Conventional Commits 格式:

```
<type>(<scope>): <Chinese subject>
```

**示例:**
- ✅ `feat(auth): 添加 OAuth 2.0 登录支持`
- ✅ `fix(api): 修复用户列表分页问题`

---

## PR 描述必须包含

### 1. 中文总结

清晰描述:
- 这个 PR 做了什么?
- 为什么需要这个变更?
- 解决了什么问题?

### 2. 相关 Issue 链接

```markdown
Closes #123
Related to #456
```

### 3. 验证步骤

#### 自动化检查
- [ ] `pnpm lint` 通过
- [ ] `pnpm format` 通过
- [ ] `pnpm typecheck` 通过
- [ ] `pnpm test` 通过

#### 手动测试
1. 具体测试步骤
2. 预期结果
3. 测试环境信息

### 4. 影响评估

包含:
- 数据库迁移需求
- 配置变更
- API 变更
- 向后兼容性

---

## PR 模板示例

```markdown
## 总结

[中文描述主要变更和目的]

## 相关 Issues

Closes #[issue编号]

## 主要变更

- [变更1]
- [变更2]

## 验证步骤

### 自动化检查
- [ ] `pnpm lint` 通过
- [ ] `pnpm format` 通过
- [ ] `pnpm typecheck` 通过
- [ ] `pnpm test` 通过

### 手动测试
1. [测试步骤1]
2. [测试步骤2]

## 影响评估

### 数据库迁移
- [ ] 需要迁移 / [x] 无需迁移

### 配置变更
- [ ] 新增环境变量 / [x] 无配置变更

### API 变更
- [ ] Breaking changes / [x] 向后兼容
```

---

## 审查前检查清单

- [ ] 本地 lint 通过
- [ ] 代码格式化完成
- [ ] 类型检查通过
- [ ] 测试通过
- [ ] 无类型断言 (除非有正当理由并documented)
- [ ] Commit 消息遵循规范
- [ ] 无 AI 标识
- [ ] 手动测试完成
- [ ] 文档已更新 (如有必要)

---

## 代码审查指南

审查者关注点:
1. **功能正确性**: 代码是否实现预期功能?
2. **代码质量**: 是否遵循项目标准和最佳实践?
3. **测试覆盖**: 是否有充分的测试?
4. **性能**: 是否有性能问题?
5. **安全**: 是否有安全漏洞?
6. **可维护性**: 代码是否易于理解和维护?

---

## 使用例子

### 创建 PR

```bash
# 1. 确保所有质量检查通过
pnpm format && pnpm lint && pnpm typecheck && pnpm test

# 2. 提交changes
git add .
git commit -m "feat(auth): 添加 OAuth 2.0 支持"

# 3. 推送到远程
git push origin feature/oauth-support

# 4. 创建 PR (使用 gh CLI)
gh pr create --title "feat(auth): 添加 OAuth 2.0 登录支持" --body "$(cat <<'EOF'
## 总结

实现 OAuth 2.0 登录功能,支持 Google 和 GitHub 第三方登录。

## 相关 Issues

Closes #123

## 主要变更

- 添加 OAuth 2.0 客户端配置
- 实现授权流程和回调处理
- 添加用户账户关联逻辑
- 更新登录 UI 添加第三方登录按钮

## 验证步骤

### 自动化检查
- [x] `pnpm lint` 通过
- [x] `pnpm format` 通过
- [x] `pnpm typecheck` 通过
- [x] `pnpm test` 通过

### 手动测试
1. 点击"使用 Google 登录"按钮
2. 完成 Google OAuth 授权流程
3. 验证成功登录并创建用户账户
4. 测试 GitHub 登录流程

## 影响评估

### 数据库迁移
- [x] 需要迁移 - 添加 oauth_accounts 表

### 配置变更
- [x] 新增环境变量:
  - GOOGLE_CLIENT_ID
  - GOOGLE_CLIENT_SECRET
  - GITHUB_CLIENT_ID
  - GITHUB_CLIENT_SECRET

### API 变更
- [x] 向后兼容 - 新增端点,不影响现有 API
