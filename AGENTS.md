# AGENTS.md

## 项目概览

这是一个使用 Hexo 8 构建的中文个人博客，主题为 Butterfly 5.6，站点发布到 GitHub Pages。

- 站点配置：`_config.yml`
- Butterfly 主题配置：`_config.butterfly.yml`
- 文章源码：`source/_posts/`
- Hexo 脚手架：`scaffolds/`
- 依赖与命令：`package.json`、`package-lock.json`
- 生成产物：`public/`（已忽略，不提交）
- 部署工作区：`.deploy_git/`（已忽略，由 Hexo 管理）

## 开发环境与命令

使用 npm，并保留 `package-lock.json`。Hexo 8 声明要求 Node.js `>=20.19.0`；新环境和 CI 应使用满足该要求的 Node.js 版本。

```bash
npm install          # 安装依赖；CI 或完全可复现安装优先使用 npm ci
npm run server       # 本地预览，默认地址 http://localhost:4000
npm run build        # 生成静态站点到 public/
npm run clean        # 清理 Hexo 缓存和生成产物
```

修改文章、配置或依赖后，至少运行：

```bash
npm run build
```

若结果疑似受旧缓存影响，再运行：

```bash
npm run clean
npm run build
```

项目当前没有自动化测试、lint 或格式化脚本，不要声称这些检查已通过。验证以 Hexo 构建成功、无新增警告以及必要的本地页面检查为准。

## 内容编辑约定

- 博客正文主要使用简体中文；保持现有文章的语气和 Markdown 风格。
- 新文章放在 `source/_posts/<slug>.md`，并使用 YAML front matter。
- 正式文章通常包含 `title`、`date`、`tags` 和 `categories`；日期格式使用 `YYYY-MM-DD HH:mm:ss`，时区为 `Asia/Shanghai`。
- `tags` 和 `categories` 使用 YAML 列表，不要无故改写既有文章的元数据或发布日期。
- 项目启用了 `post_asset_folder: true`。文章图片放入与文章同名的资源目录，并优先使用 `{% asset_img filename alt %}` 引用，以确保生成后的路径正确。
- 修改教程中的命令、分支或部署说明时，应同时核对 `_config.yml` 和 `package.json`，避免文档与实际配置不一致。

## 配置与代码修改约定

- 站点级设置修改 `_config.yml`；Butterfly 外观、搜索、评论和统计功能修改 `_config.butterfly.yml`。
- 当前主题通过 npm 依赖 `hexo-theme-butterfly` 提供；不要假设 `themes/` 中存在完整主题源码，也不要直接修改 `node_modules/`。
- 添加或升级依赖时同步提交 `package.json` 和 `package-lock.json`，并确认其 Node.js 版本要求。
- 保持改动聚焦，不要顺手重排大型 YAML 配置或批量格式化文章。
- 不要提交 `public/`、`.deploy_git/`、`node_modules/`、`db.json` 或日志文件。
- 不要把密钥、访问令牌或其他凭据写入文章、配置或 Git 历史。

## 部署安全

`npm run deploy` 会通过 SSH 将生成的网站推送到 `WendyZhou2021/WendyZhou2021.github.io.git` 的 `main` 分支，属于会改变远程状态的操作。

- 只有用户明确要求发布或部署时才运行 `npm run deploy`。
- 部署前先运行 `npm run clean && npm run build`，确认生成成功。
- 不要自行执行 `git push`、创建发布提交或更改部署仓库与分支。
- 源码与部署产物采用不同分支/工作区管理；不要把生成产物复制回源码分支。

## 完成标准

交付前应：

1. 检查 `git diff`，确认只包含任务相关改动。
2. 对文章 front matter、内部链接、图片文件名和配置缩进做人工检查。
3. 对文章、配置或依赖变更运行 `npm run build`；若未运行或无法运行，应明确说明原因。
4. 汇报修改的文件、验证命令与结果，并指出任何仍需用户确认的发布步骤。
