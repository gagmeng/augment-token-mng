# 工作流迁移指南

## 快速迁移步骤

### 1. 启用新工作流

新的自动发布工作流已创建在：
```
.github/workflows/auto-release.yml
```

该工作流会在以下情况自动运行：
- 推送代码到 main/master 分支
- 手动触发（在 Actions 页面）

### 2. 清理旧工作流（可选）

以下旧工作流的功能已整合到新工作流中，可以安全删除：

#### 可以删除的文件：

```bash
# 删除已整合的工作流
rm .github/workflows/build.yml
rm .github/workflows/version-bump-and-tag.yml
rm .github/workflows/notify-telegram.yml
rm .github/workflows/package-dispatch.yml
```

#### 可以保留的文件：

```bash
# 保留用于手动测试构建
.github/workflows/manual-build.yml
```

### 3. 验证配置

确保以下 GitHub Secrets 已配置：

**必需：**
- `GITHUB_TOKEN` ✅ （GitHub 自动提供）

**可选：**
- `TAP_DISPATCH_TOKEN` - 用于通知 Homebrew/Scoop 更新

### 4. 测试新工作流

#### 方法 1：手动触发测试
1. 进入 GitHub 仓库的 Actions 页面
2. 选择 "Auto Release" 工作流
3. 点击 "Run workflow"
4. 选择 `patch` 版本类型
5. 点击 "Run workflow" 按钮

#### 方法 2：推送代码测试
```bash
# 做一个小改动
echo "# Test" >> README.md
git add README.md
git commit -m "test: 测试自动发布工作流"
git push origin main
```

## 功能对比表

| 功能 | 旧工作流 | 新工作流 |
|------|---------|---------|
| 版本升级 | 手动运行 `version-bump-and-tag.yml` | ✅ 自动或手动 |
| 创建 Tag | 手动或通过版本升级工作流 | ✅ 自动创建 |
| 多平台构建 | `build.yml` 在 tag 推送时触发 | ✅ 自动触发 |
| 生成 Changelog | ✅ 在构建时生成 | ✅ 在构建时生成 |
| 创建 Release | ✅ 自动创建 | ✅ 自动创建 |
| Telegram 通知 | ✅ 构建完成后通知 | ❌ 已移除 |
| 通知包管理器 | ✅ Release 后触发 | ✅ 构建完成后通知 |
| 手动构建测试 | ✅ `manual-build.yml` | ✅ 保留原工作流 |

## 工作流程对比

### 旧流程（3-4 步）
```
1. 手动运行 version-bump-and-tag.yml
   ↓
2. 自动触发 build.yml（检测到新 tag）
   ↓
3. 自动触发 notify-telegram.yml（构建完成）
   ↓
4. 自动触发 package-dispatch.yml（Release 创建）
```

### 新流程（1 步）
```
推送代码或手动触发
   ↓
auto-release.yml 自动完成所有步骤：
  - 版本升级
  - 创建 tag
  - 多平台构建
  - 创建 Release
  - 通知包管理器
```

## 常见问题

### Q: 旧的 tag 和 Release 会受影响吗？
A: 不会。新工作流只处理新的发布，不会修改历史记录。

### Q: 我还能手动控制版本号吗？
A: 可以。在 Actions 页面手动触发工作流，选择版本类型或输入自定义版本号。

### Q: 如果我不想自动发布怎么办？
A: 在提交信息中添加 `[skip ci]`，例如：
```bash
git commit -m "docs: 更新文档 [skip ci]"
```

### Q: Telegram 通知功能还能用吗？
A: 已完全移除。如需通知，建议使用 GitHub 的通知功能或配置其他第三方集成。

### Q: 构建失败会怎样？
A: 每个平台独立构建，一个失败不影响其他平台。失败的平台可以手动重试。

### Q: 如何回滚到旧工作流？
A: 简单恢复旧的工作流文件即可：
```bash
git checkout HEAD~1 .github/workflows/
git commit -m "revert: 恢复旧工作流"
git push
```

## 清理命令

如果确认新工作流运行正常，可以执行以下命令清理旧文件：

```bash
# 删除旧工作流
git rm .github/workflows/build.yml
git rm .github/workflows/version-bump-and-tag.yml
git rm .github/workflows/notify-telegram.yml
git rm .github/workflows/package-dispatch.yml

# 提交更改
git commit -m "chore: 清理旧的工作流文件"
git push origin main
```

## 回滚计划

如果新工作流出现问题，可以快速回滚：

```bash
# 1. 禁用新工作流
# 在 .github/workflows/auto-release.yml 第一行添加：
# # 临时禁用

# 2. 恢复旧工作流
git revert <commit-hash>  # 回滚删除旧工作流的提交

# 3. 推送更改
git push origin main
```

## 支持

如有问题，请查看：
- [自动发布工作流程说明](./AUTO_RELEASE_WORKFLOW.md)
- [GitHub Actions 日志](../../actions)
- 提交 Issue 寻求帮助
