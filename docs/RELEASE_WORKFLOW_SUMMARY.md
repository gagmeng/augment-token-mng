# GitHub Actions 自动发布流程改进总结

## 改进目标

✅ 移除 Telegram 通知功能  
✅ 实现完全自动化的打包流程  
✅ 自动创建 tag 和 Release  
✅ 简化工作流程，减少手动操作  

## 核心改进

### 1. 统一工作流

**之前**：需要 4-5 个独立的工作流文件协同工作
- `version-bump-and-tag.yml` - 手动升级版本
- `build.yml` - 检测 tag 后构建
- `notify-telegram.yml` - 发送通知
- `package-dispatch.yml` - 通知包管理器

**现在**：只需 1 个工作流文件
- `auto-release.yml` - 完成所有任务

### 2. 自动化程度提升

| 操作 | 之前 | 现在 |
|------|------|------|
| 版本升级 | 手动触发工作流 | 推送代码自动升级 |
| 创建 Tag | 手动或通过工作流 | 自动创建 |
| 构建打包 | Tag 触发 | 自动触发 |
| 创建 Release | 自动 | 自动 |
| 通知 | Telegram | 已移除 |
| 包管理器更新 | 自动 | 自动 |

### 3. 工作流程对比

#### 旧流程（需要 3-4 步）
```
开发者操作：
1. 手动运行 version-bump-and-tag.yml
   ↓
自动流程：
2. build.yml 检测到新 tag 自动运行
   ↓
3. notify-telegram.yml 发送通知
   ↓
4. package-dispatch.yml 通知包管理器
```

#### 新流程（只需 1 步）
```
开发者操作：
1. git push origin main
   ↓
自动流程：
2. auto-release.yml 自动完成：
   - 检测代码变更
   - 升级版本号
   - 创建 tag
   - 多平台构建
   - 创建 Release
   - 通知包管理器
```

## 新工作流特性

### 智能触发机制

1. **自动触发**
   - 监听 main/master 分支的推送
   - 只在关键路径变更时触发：
     - `src/**` - 前端代码
     - `src-tauri/**` - 后端代码
     - `package.json` - 依赖配置
     - `Cargo.toml` - Rust 依赖

2. **手动触发**
   - 在 GitHub Actions 页面手动运行
   - 可选择版本类型：
     - `patch` - 修复版本（1.8.5 → 1.8.6）
     - `minor` - 次要版本（1.8.5 → 1.9.0）
     - `major` - 主要版本（1.8.5 → 2.0.0）
     - `custom` - 自定义版本（如 2.0.0-beta.1）

### 版本管理

自动更新以下文件中的版本号：
- `package.json`
- `src-tauri/Cargo.toml`
- `src-tauri/tauri.conf.json`

提交信息包含 `[skip ci]` 标记，避免循环触发。

### 多平台构建

并行构建以下平台，互不影响：
- Windows (x64) - MSI 安装包
- macOS Apple Silicon (ARM64) - DMG 镜像
- macOS Intel (x64) - DMG 镜像
- Linux (x64) - DEB 包和 AppImage

### 自动生成 Changelog

从上一个 tag 到当前 tag 的所有提交自动生成 changelog，包含：
- 提交历史
- 安装说明
- 相关链接

### 包管理器通知

构建完成后自动通知：
- Homebrew tap 仓库
- Scoop bucket 仓库

## 文件结构

### 新增文件
```
.github/workflows/
  └── auto-release.yml          # 新的统一工作流

docs/
  ├── AUTO_RELEASE_WORKFLOW.md  # 工作流使用说明
  ├── MIGRATION_GUIDE.md        # 迁移指南
  └── RELEASE_WORKFLOW_SUMMARY.md  # 本文件
```

### 可删除的旧文件
```
.github/workflows/
  ├── build.yml                 # 功能已整合
  ├── version-bump-and-tag.yml  # 功能已整合
  ├── notify-telegram.yml       # 已移除通知功能
  └── package-dispatch.yml      # 功能已整合
```

### 保留的文件
```
.github/workflows/
  └── manual-build.yml          # 用于手动测试构建
```

## 使用示例

### 日常开发（自动发布）

```bash
# 1. 修改代码
vim src/App.vue

# 2. 提交并推送
git add .
git commit -m "feat: 添加新功能"
git push origin main

# 3. 自动触发发布流程
# - 版本自动从 1.8.5 升级到 1.8.6
# - 自动创建 tag v1.8.6
# - 自动构建所有平台
# - 自动创建 GitHub Release
```

### 发布新功能（手动控制版本）

```bash
# 1. 在 GitHub Actions 页面
# 2. 选择 "Auto Release" 工作流
# 3. 点击 "Run workflow"
# 4. 选择 version_type: minor
# 5. 点击 "Run workflow" 按钮

# 结果：版本从 1.8.6 升级到 1.9.0
```

### 跳过自动发布

```bash
# 提交信息中添加 [skip ci]
git commit -m "docs: 更新文档 [skip ci]"
git push origin main

# 不会触发自动发布
```

## 配置要求

### GitHub Secrets

**必需：**
- `GITHUB_TOKEN` - GitHub 自动提供，无需配置

**可选：**
- `TAP_DISPATCH_TOKEN` - 用于通知 Homebrew 和 Scoop 仓库
  - 如果未配置，包管理器通知会跳过（非关键功能）

### 权限设置

工作流需要以下权限：
- `contents: write` - 用于创建 tag、提交代码、创建 Release

## 优势总结

### 1. 效率提升
- **之前**：需要 3-4 个手动步骤
- **现在**：推送代码即可，0 手动步骤

### 2. 错误减少
- 自动化减少人为错误
- 版本号自动同步，避免不一致
- 失败重试更简单

### 3. 流程简化
- 1 个工作流替代 4-5 个
- 更容易理解和维护
- 日志更清晰

### 4. 灵活性
- 支持自动和手动两种模式
- 可以选择版本升级类型
- 可以跳过自动发布

### 5. 可靠性
- 每个平台独立构建
- 一个失败不影响其他
- 非关键步骤失败不阻塞流程

## 迁移步骤

### 1. 立即可用
新工作流 `auto-release.yml` 已创建，可以立即使用。

### 2. 测试验证
```bash
# 方法 1：手动触发测试
# 在 GitHub Actions 页面手动运行 Auto Release

# 方法 2：推送代码测试
echo "# Test" >> README.md
git add README.md
git commit -m "test: 测试自动发布"
git push origin main
```

### 3. 清理旧文件（可选）
确认新工作流正常后，可以删除旧文件：
```bash
git rm .github/workflows/build.yml
git rm .github/workflows/version-bump-and-tag.yml
git rm .github/workflows/notify-telegram.yml
git rm .github/workflows/package-dispatch.yml
git commit -m "chore: 清理旧工作流"
git push origin main
```

## 故障排除

### 构建失败
- 查看 Actions 日志中的详细错误
- 每个平台独立，可以单独重试失败的平台

### 版本冲突
- 工作流会检测现有 tag
- 如果 tag 已存在，会跳过创建

### 包管理器通知失败
- 这是非关键错误，不影响主流程
- 检查 `TAP_DISPATCH_TOKEN` 配置

### 循环触发
- 版本提交包含 `[skip ci]` 标记
- 不会触发新的构建

## 后续优化建议

1. **添加测试步骤**
   - 在构建前运行单元测试
   - 确保代码质量

2. **添加代码签名**
   - macOS 应用签名
   - Windows 代码签名

3. **添加更新检查**
   - 应用内检查更新
   - 自动下载更新

4. **性能优化**
   - 使用构建缓存
   - 并行化更多步骤

5. **通知增强**
   - 添加 Discord/Slack 通知
   - 邮件通知选项

## 相关文档

- [自动发布工作流程说明](./AUTO_RELEASE_WORKFLOW.md) - 详细使用说明
- [工作流迁移指南](./MIGRATION_GUIDE.md) - 迁移步骤和常见问题
- [GitHub Actions 文档](https://docs.github.com/en/actions) - 官方文档

## 总结

新的自动发布工作流实现了：
- ✅ 完全自动化的版本管理
- ✅ 自动创建 tag 和 Release
- ✅ 移除 Telegram 通知
- ✅ 简化工作流程
- ✅ 提高发布效率
- ✅ 减少人为错误

开发者只需专注于代码开发，推送到主分支即可自动完成发布流程。
