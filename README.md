# 我的 Hexo 博客

基于 [Hexo](https://hexo.io/) 构建的个人博客，部署于 [Gitee Pages](https://gitee.com/)。

## 本地预览

```bash
# 安装依赖
npm install

# 启动本地服务器（默认 http://localhost:4000）
npx hexo server

# 生成静态文件
npx hexo clean && npx hexo generate
```

## 写作

```bash
# 新建文章
npx hexo new "文章标题"
```

文章位于 `source/_posts/` 目录下，使用 Markdown 格式编写。

## 部署到 Gitee Pages

```bash
# 清理、生成并部署静态文件到 Gitee 仓库的 pages 分支
npx hexo clean && npx hexo generate && npx hexo deploy
```

部署完成后，前往 Gitee 仓库的「服务 → Gitee Pages」，选择 `pages` 分支开启/更新即可。

## 目录说明

| 目录/文件 | 说明 |
| --- | --- |
| `_config.yml` | 站点配置文件 |
| `source/_posts/` | 博客文章（Markdown） |
| `themes/` | 主题文件 |
| `package.json` | 依赖与脚本 |