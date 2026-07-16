---
title: Hexo 写作与常用命令指南
date: 2026-07-16 23:50:00
tags:
  - Hexo
  - 教程
  - 写作
  - 原理
categories:
  - 技术
---

本篇是本博客的第一篇正式文章，记录使用 Hexo 写作的全部常用操作与其背后的原理，方便日后随时查阅。

## 一、创建文章

### 1. 新建一篇博客

```bash
hexo new "文章标题"
```

执行后会在 `source/_posts/` 下生成 `文章标题.md`，并附带同名的资源文件夹（用于存放图片等资源）。

### 2. 文章模板（Front-matter）

每篇文章顶部的 `---` 区块称为 **Front-matter**，用于定义元信息：

```yaml
---
title: 我的文章标题        # 文章标题
date: 2026-07-16 10:00:00 # 发布时间
tags:                     # 标签（可多个）
  - Hexo
  - 教程
categories:               # 分类（可多个）
  - 技术
---
```

### 3. 文章中插入图片（无需图床）

本项目已开启 `post_asset_folder: true`，每篇文章都有一个**同名资源文件夹**。把图片放入该文件夹后，图片会随文章一起生成、一起部署到 GitHub，**不需要任何外部图床**。

> ⚠️ 注意：标准 Markdown 语法 `![](图片.png)` 在子目录渲染时容易路径失配，**推荐使用 Hexo 专有标签**：

```nunjucks
{% asset_img example.png 图片描述 %}
```

该语法会自动生成正确路径，本地预览和发布到 GitHub Pages 后都能正常显示。

## 二、本地预览

写完文章后，启动本地服务器预览：

```bash
hexo server
# 或简写
hexo s
```

浏览器访问 `http://localhost:4000` 即可实时查看。修改文章保存后会自动刷新（部分主题需手动刷新）。**本地预览不会影响线上网站**，是"写但不发布"的最佳工具。

## 三、生成与部署

### 1. 三步发布流程

```bash
hexo clean      # 清理旧的生成文件
hexo generate   # 生成静态文件到 public/
hexo deploy     # 部署到远程仓库
```

可使用简写：`hexo clean && hexo g && hexo d`

### 2. 本项目的部署目标

当前 `_config.yml` 配置的部署目标是 GitHub Pages（用户页 `WendyZhou2021.github.io`）：

```yaml
deploy:
  type: git
  repo: git@github.com:WendyZhou2021/WendyZhou2021.github.io.git
  branch: main
```

部署后访问：<https://WendyZhou2021.github.io/>（注意 `W` 大写）。

## 四、源码与网站的双分支管理

本仓库在 GitHub 上采用**双分支**结构：

| 分支        | 内容                 | 作用                       |
| ----------- | -------------------- | -------------------------- |
| `main`      | 构建产物（HTML/CSS） | GitHub Pages 直接服务网站  |
| `source`    | 源码（Markdown 等）  | 源码备份与版本管理         |

### 完整的发布工作流

每次写完文章，按以下两步操作：

```bash
# 第一步：发布网站（更新 main 分支的构建产物）
hexo clean && hexo generate && hexo deploy

# 第二步：提交源码（更新 source 分支）
git add .
git commit -m "feat: 新文章xxx"
git push git@github.com:WendyZhou2021/WendyZhou2021.github.io.git master:source
```

> 注意：`hexo deploy` 推送的是构建产物到 `main`；源码需单独 `git commit` + `git push` 才会同步到 `source`。两者互不干扰。

### 只保存草稿、不发布的方法

如果文章还没写完只想备份，**不执行 `hexo deploy` 即可**，网站不会更新：

```bash
git add .
git commit -m "wip: 草稿"
git push git@github.com:WendyZhou2021/WendyZhou2021.github.io.git master:source
# ❌ 不要执行 hexo deploy
```

## 五、深入原理：源码与产物是如何分离的

理解这部分原理，有助于彻底掌握 Hexo 的发布机制。

### 1. 三个独立的工作区

你的项目目录里其实存在三个互相隔离的目录：

```
myblog/                    ← 工作区 1：源码（被 git 跟踪）
├── _config.yml            ← 源码配置
├── source/_posts/*.md     ← 文章源码
├── scaffolds/             ← 文章模板
└── ...

myblog/public/             ← 工作区 2：生成产物（被 .gitignore 排除）
└── index.html, css/, js/  ← hexo generate 生成的网站文件

myblog/.deploy_git/        ← 工作区 3：部署专用 git 仓库（被 .gitignore 排除）
├── .git/                  ← 独立的 git 仓库，指向 main 分支
└── index.html, css/, js/  ← 从 public 复制而来
```

### 2. 机制一：`.gitignore` 保证源码仓库干净

项目根目录的 `.gitignore` 排除了产物相关目录：

```
public/         ← 生成产物目录，被排除
.deploy*/       ← 部署目录，被排除
node_modules/   ← 依赖目录，被排除
```

因此执行 `git add .` 时**只会提交源码**，HTML/CSS/JS 等产物永远不会污染源码分支（`source`）。

### 3. 机制二：`.deploy_git` 是独立的 git 仓库

`.deploy_git/` 目录内部有**自己独立的 `.git/`**，它不归你管理，而是由 `hexo deploy` 全权负责。这就是源码与产物井水不犯河水的核心。

### 4. `hexo deploy` 内部做了什么

当你敲下 `hexo deploy`，它本质上自动执行了以下完整流程（无需手动操作）：

```bash
# 1. 准备部署目录
cd .deploy_git
git init
git checkout -b main                          # 切到 main 分支

# 2. 把产物复制进来
cp -r ../public/* .

# 3. 用你的 git 全局信息自动提交
git config user.name "WendyZhou"
git config user.email "1647534125@qq.com"
git add -A
git commit -m "Site updated: 2026-07-17 01:58:27"   # 固定格式：Site updated: 时间戳

# 4. 强制推送到远程 main 分支
git push --force git@github.com:WendyZhou2021/WendyZhou2021.github.io.git main:main
```

几个关键点：

- **自动提交**：每次 `hexo deploy` 都产生一条提交，提交信息固定为 `Site updated: 时间`。
- **强制推送（--force）**：每次部署会完全覆盖 `main` 分支，所以 main 永远是"最新的完整网站"，不会累积历史垃圾。
- **隔离运行**：所有 git 操作都在 `.deploy_git/` 内进行，**完全不碰**项目根目录的源码 git 仓库。

### 5. 完整数据流向图

```
你写文章 (source/_posts/xxx.md)
      │
      ├──→ git commit + git push master:source
      │        ↓
      │    GitHub 的 source 分支（存源码）
      │
      └──→ hexo generate
               ↓
            public/（产物，被 gitignore 忽略）
               ↓
            hexo deploy
               ↓
            .deploy_git/（独立 git 仓库，自动 commit）
               ↓
            强制推送到 GitHub 的 main 分支（存产物）
               ↓
            GitHub Pages 读取 main，对外提供网站
```

### 6. 一句话总结

**分离的本质是：两条不同的 git 推送命令、推到两个不同的分支；`.gitignore` 保证源码仓库干净，`.deploy_git` 用独立 git 仓库处理产物。** 只要你不执行 `hexo deploy`，产物分支（main）就不会动，网站就不会更新——这正是"随时保存、择机发布"的关键。

## 六、常用命令速查表

| 命令                | 简写       | 说明                       |
| ------------------- | ---------- | -------------------------- |
| `hexo new "标题"`   | -          | 新建文章                   |
| `hexo new page "x"` | -          | 新建页面（如关于页）       |
| `hexo server`       | `hexo s`   | 启动本地预览（端口 4000）  |
| `hexo generate`     | `hexo g`   | 生成静态文件               |
| `hexo deploy`       | `hexo d`   | 部署到远程仓库             |
| `hexo clean`        | -          | 清理缓存与 public 目录     |
| `hexo list post`    | -          | 列出所有文章               |

## 七、Markdown 写作要点

### 1. 标题层级

```markdown
## 二级标题
### 三级标题
```

### 2. 代码块

````markdown
```javascript
const x = 1;
```
````

支持的语言高亮包括：`javascript`、`python`、`bash`、`yaml`、`json`、`html`、`css` 等。

### 3. 引用与提示

```markdown
> 这是一段引用文字
```

### 4. 列表

```markdown
- 无序列表项
  - 嵌套项

1. 有序列表第一项
2. 有序列表第二项
```

### 5. 链接与图片

```markdown
[链接文字](https://example.com)
```

图片请使用上文提到的 `{% asset_img 文件名 描述 %}` 语法。

## 八、常用目录说明

```
myblog/
├── _config.yml              # 站点配置文件（部署目标等）
├── _config.butterfly.yml    # 主题配置文件（评论、统计、外观）
├── package.json             # 依赖定义
├── source/
│   └── _posts/              # ★ 所有文章都在这里
├── scaffolds/               # 文章模板
├── themes/                  # 主题文件
├── public/                  # 生成产物（被 gitignore，不提交）
└── .deploy_git/             # 部署专用仓库（被 gitignore，不提交）
```

## 九、本博客已接入的功能

- 🎨 **主题**：Butterfly 5.6.0（现代美观，功能丰富）
- 💬 **评论**：Giscus（基于 GitHub Discussions，零成本接入）
- 📊 **统计**：不蒜子（显示访问量/访客数，主题内置）
- 🚀 **托管**：GitHub Pages（用户页 `WendyZhou2021.github.io`）

评论管理地址：<https://github.com/WendyZhou2021/WendyZhou2021.github.io/discussions>

## 十、后续进阶

- 🖼️ 自定义头像/背景：修改 `_config.butterfly.yml` 的 `avatar`、`background`
- 🔍 启用本地搜索：在 `_config.butterfly.yml` 配置 `search.use: local_search`
- 📝 接入文章分类与标签页：使用 `hexo new page tags` / `hexo new page categories`
- 🌐 绑定自定义域名：在 GitHub Pages 设置中配置
- ⚡ 启用 PWA / CDN 加速：参考 Butterfly 官方文档

---

> 📖 更多内容请参考 [Hexo 官方文档](https://hexo.io/zh-cn/docs/) 与 [Butterfly 文档](https://butterfly.js.org/)