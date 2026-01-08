# 贡献指南

感谢你考虑为 CloudBase AI ToolKit 做出贡献！在提交贡献之前，请花点时间阅读以下指南。

## 项目安装

1. 克隆项目
```bash
git clone https://github.com/TencentCloudBase/CloudBase-AI-ToolKit.git
cd CloudBase-AI-ToolKit
```

2. 安装依赖
```bash
# 使用 npm
npm install

# 或使用 yarn
yarn install

# 或使用 pnpm
pnpm install
```

## 开发流程

1. Fork 本仓库
2. 创建你的特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交你的改动 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 打开一个 Pull Request

## 提交规范

为了自动生成 changelog，请遵循以下提交规范：

- `feat`: ✨ 新功能
- `fix`: 🐛 修复 bug
- `docs`: 📝 文档更新
- `style`: 💄 代码格式（不影响代码运行的变动）
- `refactor`: ♻️ 重构（既不是新增功能，也不是修改 bug 的代码变动）
- `perf`: ⚡ 性能优化
- `test`: ✅ 增加测试
- `chore`: 🔧 构建过程或辅助工具的变动

提交示例：
```bash
git commit -m "feat: 添加自动生成 changelog 功能"
git commit -m "fix: 修复部署失败的问题"
git commit -m "docs: 更新 README 文档"
```

## 版本管理

项目使用 standard-version 进行版本管理，支持以下版本类型：

- 正式版本：`npm run release`
- Alpha 版本：`npm run release:alpha`
- Beta 版本：`npm run release:beta`
- RC 版本：`npm run release:rc`

版本号规则：
- 主版本号：不兼容的 API 修改
- 次版本号：向下兼容的功能性新增
- 修订号：向下兼容的问题修正

预发布版本号规则：
- alpha: 内部测试版本
- beta: 公测版本
- rc: 候选发布版本

## Changelog 生成

项目使用 conventional-changelog 自动生成 changelog：

1. 首次生成（包含所有历史记录）：
```bash
npm run changelog:first
```

2. 生成新的变更记录：
```bash
npm run changelog
```

生成的 changelog 将保存在 `CHANGELOG.md` 文件中。

## Rules 管理流程

项目使用硬链接机制来保持所有 AI IDE 配置文件的一致性。理解这个流程对于正确维护 rules 非常重要。

### 核心原则

1. **新增模块 rules**：在 `config/.claude/skills` 中创建
2. **系统顶级 rules**：在 `config/.cursor/rules/cloudbase-rules.mdc` 中修改

### 目录结构关系

```
config/
├── .claude/
│   └── skills/              # 源目录：新增模块 rules 在这里创建
│       └── [module]/
│           └── SKILL.md     # 模块规则文件
├── .codebuddy/
│   └── skills/              # 硬链接自 .claude/skills
├── rules/                   # 从 .claude/skills 复制（SKILL.md -> rule.md）
│   └── [module]/
│       └── rule.md
└── .cursor/
    └── rules/
        ├── cloudbase-rules.mdc  # 系统顶级 rules（在这里修改）
        └── [module]/            # 硬链接自 config/rules（.md -> .mdc 转换）
```

### 同步流程

运行 `npm run build:rules-sync` 或 `node scripts/fix-config-hardlinks.mjs` 会执行以下同步：

1. **Skills 目录同步**（硬链接）
   - `config/.claude/skills` → `config/.codebuddy/skills`
   - 使用硬链接，修改任一文件会同步到另一个

2. **Skills 到 Rules 同步**（复制）
   - `config/.claude/skills` → `config/rules`
   - 复制文件，`SKILL.md` 重命名为 `rule.md`
   - 保持原有目录结构

3. **Rules 到 IDE 目录同步**（硬链接）
   - `config/rules` → 各 IDE 特定目录：
     - `config/.qoder/rules`
     - `config/.cursor/rules`（.md → .mdc 转换）
     - `config/.agent/rules`
     - `config/.trae/rules`
     - `config/.windsurf/rules`
     - `config/.clinerules`
     - `config/.kiro/steering`
   - 使用硬链接，Cursor 需要将 `.md` 转换为 `.mdc` 格式

4. **系统顶级 Rules 同步**（硬链接）
   - `config/.cursor/rules/cloudbase-rules.mdc` → 其他 IDE 的规则文件
   - 使用硬链接，修改 Cursor 的文件会同步到所有其他 IDE

### 如何新增模块 Rules

1. **在 `config/.claude/skills` 中创建新模块**：
   ```bash
   config/.claude/skills/[module-name]/
   └── SKILL.md
   ```

2. **运行同步脚本**：
   ```bash
   npm run build:rules-sync
   ```

3. **自动同步结果**：
   - ✅ 自动复制到 `config/rules/[module-name]/rule.md`
   - ✅ 自动硬链接到所有 IDE 目录
   - ✅ Cursor 目录会自动转换为 `.mdc` 格式

### 如何修改系统顶级 Rules

1. **直接编辑 `config/.cursor/rules/cloudbase-rules.mdc`**

2. **运行同步脚本**：
   ```bash
   npm run build:rules-sync
   ```

3. **自动同步结果**：
   - ✅ 通过硬链接自动同步到所有其他 IDE 的规则文件
   - ✅ 修改一个文件，所有文件都会更新（因为是硬链接）

### 重要提示

- ⚠️ **不要直接修改** `config/rules` 或其他 IDE 目录中的文件
- ✅ **新增模块**：在 `config/.claude/skills` 中创建
- ✅ **修改顶级规则**：在 `config/.cursor/rules/cloudbase-rules.mdc` 中修改
- ✅ **运行同步**：修改后运行 `npm run build:rules-sync` 同步到所有位置
- 💡 **硬链接机制**：使用硬链接确保所有 IDE 配置文件保持一致，修改一处即可同步到所有位置

## 代码风格

- 遵循项目的代码风格指南
- 确保所有测试通过

## 提交 Pull Request

1. 确保你的 PR 描述清晰地说明了变更内容
2. 如果可能，添加相关的测试用例
3. 确保你的代码符合项目的代码风格
4. 更新相关文档

## 问题反馈

如果你发现任何问题或有改进建议，请：

1. 使用 GitHub Issues 提交问题
2. 提供详细的问题描述和复现步骤
3. 如果可能，提供相关的代码示例

## GitHub Actions Workflows

项目提供了两个 GitHub Actions workflow 用于自动化配置同步和构建：

### Build Example Zips

**Workflow**: `.github/workflows/build-zips.yml`

用于同步配置到 cloudbase-examples 仓库并构建 zip 文件。

**使用场景**：
- 需要构建示例模板的 zip 文件
- 需要将构建产物发布为 artifact 供内网系统拉取

**参数**：
- `source_branch`: 源分支（本仓库的分支，默认：main）
- `target_branch`: 目标分支（cloudbase-examples 的分支，默认：master）
- `build_zips`: 是否构建 zip 文件（默认：true）
- `commit_changes`: 是否提交更改到 cloudbase-examples（默认：false）

**使用方法**：
1. 在 GitHub 仓库的 Actions 页面选择 "Build Example Zips"
2. 点击 "Run workflow"
3. 填写参数（可选）
4. 点击 "Run workflow" 按钮

**输出**：
- 如果 `build_zips` 为 true，会在 Actions 页面生成 artifact `cloudbase-examples-zips`，保留 30 天

### Sync Branch to Examples

**Workflow**: `.github/workflows/sync-branch.yml`

仅用于将本仓库的指定分支同步到 cloudbase-examples 的指定分支，不构建 zip 文件。

**使用场景**：
- 需要将某个分支的配置同步到 cloudbase-examples 的对应分支
- 不需要构建 zip 文件

**参数**：
- `source_branch`: 源分支（本仓库的分支，必填）
- `target_branch`: 目标分支（cloudbase-examples 的分支，必填）
- `commit_changes`: 是否提交更改到 cloudbase-examples（默认：true）

**使用方法**：
1. 在 GitHub 仓库的 Actions 页面选择 "Sync Branch to Examples"
2. 点击 "Run workflow"
3. 填写必填参数：`source_branch` 和 `target_branch`
4. 选择是否提交更改
5. 点击 "Run workflow" 按钮

**注意事项**：
- 由于两个仓库在同一组织下，使用默认的 `GITHUB_TOKEN` 即可访问
- 如果目标分支不存在，workflow 会自动创建

## 行为准则

- 尊重所有贡献者
- 接受建设性的批评
- 关注问题本身

感谢你的贡献！ 