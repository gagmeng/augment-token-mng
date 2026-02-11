# 自动发布工作流程说明

## 概述

新的自动发布工作流程实现了完全自动化的版本管理、打包和发布流程，无需手动干预。

## 触发方式

### 1. 自动触发（推荐）

当代码推送到 `main` 或 `master` 分支，且包含以下路径的更改时自动触发：
- `src/**` - 前端代码
- `src-tauri/**` - Tauri 后端代码
- `package.json` - 依赖配置
- `Cargo.toml` - Rust 依赖

**自动行为：**
- 自动升级 patch 版本（例如：1.8.5 → 1.8.6）
- 自动创建 tag
- 自动打包所有平台
- 自动创建 GitHub Release

### 2. 手动触发

在 GitHub Actions 页面手动触发，可以选择版本升级类型：
- **patch**: 1.8.5 → 1.8.6（修复bug）
- **minor**: 1.8.5 → 1.9.0（新功能）
- **major**: 1.8.5 → 2.0.0（重大更新）
- **custom**: 自定义版本号（例如：2.0.0-beta.1）

## 工作流程步骤

### 第一阶段：版本管理
1. 检查是否需要发布（检测代码变更）
2. 计算新版本号
3. 更新版本文件：
   - `package.json`
   - `src-tauri/Cargo.toml`
   - `src-tauri/tauri.conf.json`
4. 提交版本更改（带 `[skip ci]` 标记避免循环触发）
5. 创建并推送 git tag

### 第二阶段：生成变更日志
1. 从上一个 tag 到当前 tag 的所有提交
2. 生成格式化的 changelog
3. 包含安装说明和相关链接

### 第三阶段：多平台构建
并行构建以下平台：
- **Windows** (x64) - MSI 安装包
- **macOS Apple Silicon** (ARM64) - DMG 镜像
- **macOS Intel** (x64) - DMG 镜像
- **Linux** (x64) - DEB 包和 AppImage

### 第四阶段：发布
1. 创建 GitHub Release
2. 上传所有平台的安装包
3. 附加自动生成的 changelog
4. 通知包管理器（Homebrew、Scoop）

## 与旧工作流的区别

### 移除的功能
- ❌ Telegram 通知（已完全移除）
- ❌ 手动版本升级工作流（已整合）
- ❌ 独立的 tag 创建流程（已自动化）

### 新增的功能
- ✅ 智能检测代码变更
- ✅ 自动版本升级
- ✅ 统一的发布流程
- ✅ 更清晰的日志输出
- ✅ 失败时不影响其他平台构建

### 保留的功能
- ✅ 多平台并行构建
- ✅ 自动生成 changelog
- ✅ 通知包管理器更新
- ✅ 手动触发选项

## 配置要求

### 必需的 Secrets
- `GITHUB_TOKEN` - 自动提供，用于创建 Release

### 可选的 Secrets
- `TAP_DISPATCH_TOKEN` - 用于通知 Homebrew 和 Scoop 仓库更新

## 使用示例

### 场景 1：日常开发
```bash
# 修复 bug
git add .
git commit -m "fix: 修复登录问题"
git push origin main

# 自动触发：版本 1.8.5 → 1.8.6
# 自动创建 tag: v1.8.6
# 自动构建并发布
```

### 场景 2：发布新功能
```bash
# 在 GitHub Actions 页面手动触发
# 选择 version_type: minor
# 结果：版本 1.8.6 → 1.9.0
```

### 场景 3：重大版本更新
```bash
# 在 GitHub Actions 页面手动触发
# 选择 version_type: major
# 结果：版本 1.9.0 → 2.0.0
```

### 场景 4：自定义版本
```bash
# 在 GitHub Actions 页面手动触发
# 选择 version_type: custom
# 输入 custom_version: 2.0.0-beta.1
# 结果：版本 → 2.0.0-beta.1
```

## 故障排除

### 构建失败
- 检查 Actions 日志中的具体错误
- 每个平台独立构建，一个失败不影响其他平台

### 版本冲突
- 工作流会自动检测现有 tag
- 如果 tag 已存在，会跳过创建

### 包管理器通知失败
- 这是非关键错误，不会影响主发布流程
- 检查 `TAP_DISPATCH_TOKEN` 是否配置正确

## 迁移说明

旧的工作流文件可以保留或删除：
- `build.yml` - 可以删除（功能已整合）
- `version-bump-and-tag.yml` - 可以删除（功能已整合）
- `notify-telegram.yml` - 可以删除（已移除通知功能）
- `package-dispatch.yml` - 可以删除（功能已整合）
- `manual-build.yml` - 可以保留（用于测试构建）

## 优势

1. **完全自动化** - 推送代码即可自动发布
2. **版本管理简化** - 无需手动修改版本号
3. **流程统一** - 一个工作流处理所有事情
4. **更快的发布** - 并行构建节省时间
5. **更少的错误** - 减少手动操作
6. **清晰的日志** - 每个步骤都有明确的输出

## 注意事项

1. 提交信息中包含 `[skip ci]` 会跳过自动构建
2. 只有影响 `src/`、`src-tauri/` 等关键路径的更改才会触发发布
3. 版本号遵循语义化版本规范（Semantic Versioning）
4. 所有版本更改都会自动提交到仓库
