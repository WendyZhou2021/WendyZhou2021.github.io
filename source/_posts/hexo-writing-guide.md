---
title: Hexo 写作与常用命令指南
date: 2026-07-16 23:50:00
tags:
  - Hexo
  - 教程
  - 写作
categories:
  - 技术
---

本篇是本博客的第一篇正式文章，记录使用 Hexo 写作的全部常用操作，方便日后随时查阅。

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

浏览器访问 `http://localhost:4000` 即可实时查看。修改文章保存后会自动刷新（部分主题需手动刷新）。

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
git push origin source
```

> 注意：`hexo deploy` 会推送构建产物；源码需单独 `git commit` + `git push` 才会同步。

## 五、常用命令速查表

| 命令                | 简写       | 说明                       |
| ------------------- | ---------- | -------------------------- |
| `hexo new "标题"`   | -          | 新建文章                   |
| `hexo new page "x"` | -          | 新建页面（如关于页）       |
| `hexo server`       | `hexo s`   | 启动本地预览（端口 4000）  |
| `hexo generate`     | `hexo g`   | 生成静态文件               |
| `hexo deploy`       | `hexo d`   | 部署到远程仓库             |
| `hexo clean`        | -          | 清理缓存与 public 目录     |
| `hexo list post`    | -          | 列出所有文章               |

## 六、Markdown 写作要点

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

## 七、常用目录说明

```
myblog/
├── _config.yml          # 站点配置文件
├── package.json         # 依赖定义
├── source/
│   └── _posts/          # ★ 所有文章都在这里
├── scaffolds/           # 文章模板
├── themes/              # 主题文件
└── public/              # 生成产物（被 gitignore，不提交）
```

## 八、后续进阶

- 🎨 更换主题：[Hexo 主题市场](https://hexo.io/themes/)
- 🔍 启用搜索：安装 `hexo-generator-searchdb`
- 💬 接入评论：如 Giscus、Waline
- 📊 接入统计：Google Analytics、百度统计
- 🌐 多语言：参考 Hexo i18n 文档

---

> 📖 更多内容请参考 [Hexo 官方文档](https://hexo.io/zh-cn/docs/)