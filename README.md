# bd4wxr's blog

基于 [Hexo](https://hexo.io) 的静态博客，部署在 GitHub Pages，绑定自定义域名 [https://ark.bd4wxr.top](https://ark.bd4wxr.top)。

- 博客框架：Hexo 7.3.0
- 主题：`themes/arknights`（自定义明日方舟主题，来自 [Yue-plus/hexo-theme-arknights](https://github.com/Yue-plus/hexo-theme-arknights)）
- 分支结构：
  - `source`：Hexo 源码（本仓库默认分支，日常维护在此）
  - 部署方式：推送 `source` 后由 GitHub Actions 自动构建并部署到 Pages（见 `.github/workflows/deploy.yml`），无需在本地执行部署命令

## 目录结构

```
.
├── _config.yml            # Hexo 主配置（含 feed 等）
├── _config.arknights.yml  # 主题配置
├── .github/workflows/     # GitHub Actions 部署工作流
├── package.json           # 依赖与 npm 脚本
├── scaffolds/             # 新建文章的模板
├── source/
│   ├── _posts/            # 文章目录（Markdown）
│   ├── images/            # 文章图片
│   ├── CNAME              # 自定义域名
│   └── .nojekyll          # 让 GitHub 跳过 Jekyll 直接发布静态文件
└── themes/arknights/      # 主题（已入库，普通目录管理，直接替换文件即可更新）
```

## 环境要求

- Node.js（建议 18+）
- 首次使用先安装依赖：

```bash
npm install
```

> 如果 `npm install` 报 `Your cache folder contains root-owned files`，是 `~/.npm` 权限问题：
> - 临时绕过：`npm install --cache /tmp/npm-cache-xzy`
> - 永久修复：`sudo chown -R 501:20 ~/.npm`

## 日常操作

### 新建文章

**方式一：在线编辑（推荐，零配置）**

在浏览器打开 [github.dev](https://github.dev/xingzhiyou/xingzhiyou.github.io)（或仓库页面按 `.` 键），
在 `source/_posts/` 下新建 `.md` 文件，写完在左侧源代码管理面板提交并推送，CI 自动部署。

**方式二：本地命令行**

```bash
# 自动套用 scaffolds/post.md 模板生成
npx hexo new "文章标题"

# 或直接在 source/_posts/ 下手动创建 .md 文件
```

文章 front matter 格式：

```markdown
---
title: 文章标题
date: 2026-08-18 12:00:00
tags: [标签1, 标签2]
categories: [分类]
permalink: 2026/08/18/文章标题/   # 可选，不填则用默认链接格式
---

第一段内容…

<!-- more -->   <!-- 之前的部分作为首页摘要 -->

## 正文小节
```

> **重要规则**
> 1. 文章必须放在 `source/_posts/` 下（可按分类建子目录，如 `source/_posts/草籽杯/`）。放在其他位置会被当作"页面"，不会出现在首页/归档/订阅里
> 2. `date` 是文章发布与排序依据，不要填未来的时间（CI 构建时区为 UTC）
> 3. 写完后 `git push origin source` 即自动部署上线，无需其他操作

### 修改 / 删除文章

- 修改：直接编辑 `source/_posts/文章名.md` 后提交推送
- 删除：删除对应文件即可（上线后旧链接会 404）

### 添加附件（图片 / 其他文件）

**图片**：放在 `source/images/` 下（建议按分类建子目录，如 `source/images/草籽杯/`），文章中用相对路径引用（`_posts` 里的文件需回退一级）：

```markdown
![说明](../images/草籽杯/示例1.png)
```

**其他附件**（zip / pdf / 音频等）：推荐在 `source/` 下建 `files/` 目录存放。`source/` 下除 `_posts` 外的文件会被原样复制到站点根目录，附件即可通过 `/files/xxx.zip` 访问：

```markdown
[下载附件](../files/示例.zip)
```

> 附件限制：GitHub 单文件上限 100MB（建议 50MB 内），大文件请改用外部网盘/图床链接。
> 在 github.dev 里可以把图片/附件直接**拖进资源管理器**的对应目录，提交推送后即随站点上线。

### 发布上线

文章改完提交并推送到 `source` 分支后，GitHub Actions 自动构建部署（约 1-2 分钟生效）：

```bash
git add -A
git commit -m "更新说明"
git push origin source
```

查看进度：仓库 Actions 页面看最新一次 run；上线后在 `https://ark.bd4wxr.top` 确认。

### 本地预览

```bash
npm run server     # 打开 http://localhost:4000
```

## RSS 订阅

- 订阅地址：`https://ark.bd4wxr.top/atom.xml`（同时生成 `rss.xml`）
- 由 `hexo-generator-feed` 生成，配置在 `_config.yml` 的 `feed:` 段（当前输出全文、最近 20 篇）
- RSS 是**拉取式**：阅读器按周期（15 分钟～1 小时）抓取 feed，新文章最长延迟约 1 小时，手动刷新订阅可立即看到
- 想让订阅者实时收到更新，可配置 WebSub hub（见 `hexo-generator-feed` 文档）

## 常见问题

### 1. 部署后线上没更新

1. 先看 [GitHub 状态页](https://www.githubstatus.com) —— 若 Pages 处于故障/降级（429/503），等恢复后在 Actions 页面点 **Re-run jobs**
2. 检查最近一次部署是否成功：Actions 页面看最新 run，失败的通常卡在 `build` 步骤（点开日志排查，如依赖、语法问题）
3. 若一切正常但线上仍旧内容，多为浏览器/CDN 缓存，硬刷新（`Cmd+Shift+R`）即可

### 2. RSS 没推送

- 不是故障：阅读器有轮询周期，手动刷新即可
- 确认订阅的是 `https://ark.bd4wxr.top/atom.xml`

### 3. 更新主题

主题（`themes/arknights`）已转为**普通目录管理**并入库，clone 后自带主题，无需额外恢复。更新到上游最新版：

```bash
git clone https://github.com/Yue-plus/hexo-theme-arknights.git /tmp/arknights-theme
rsync -a --exclude='.git' /tmp/arknights-theme/ themes/arknights/
# 预览确认正常后（推送即自动部署）：
npm run build
git add themes/arknights && git commit -m "chore: 更新主题" && git push origin source
```

> 注意：上游更新可能改动主题配置项（如 2026-07 版移除了 busuanzi、新增 bgm/vercount），更新后检查根目录 `_config.arknights.yml` 的覆盖项是否仍然有效。

### 4. 让某篇文章重新"推送"一次给订阅者

修改该文章的 `date` 字段再重新发布即可（注意会同时改变归档/首页排序，非测试用途不建议频繁改）。

## npm 脚本

| 命令 | 作用 |
|---|---|
| `npm run clean` | `hexo clean`，清理 public 和数据库 |
| `npm run build` | `hexo generate && cp source/.nojekyll public/.nojekyll` |
| `npm run server` | `hexo server`，本地预览 |
