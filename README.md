# bd4wxr's blog

基于 [Hexo](https://hexo.io) 的静态博客，部署在 GitHub Pages，绑定自定义域名 [https://ark.bd4wxr.top](https://ark.bd4wxr.top)。

- 博客框架：Hexo 7.3.0
- 主题：`themes/arknights`（自定义明日方舟主题，来自 [Yue-plus/hexo-theme-arknights](https://github.com/Yue-plus/hexo-theme-arknights)）
- 分支结构：
  - `source`：Hexo 源码（本仓库默认分支，日常维护在此）
  - `gh-pages`：生成的静态站点（由 `hexo deploy` 推送，GitHub 自动构建部署）

## 目录结构

```
.
├── _config.yml            # Hexo 主配置（含 feed、deploy 配置）
├── _config.arknights.yml  # 主题配置
├── package.json           # 依赖与 npm 脚本
├── scaffolds/             # 新建文章的模板
├── source/
│   ├── _posts/            # 文章目录（Markdown）
│   ├── images/            # 文章图片
│   ├── CNAME              # 自定义域名
│   └── .nojekyll          # 让 GitHub 跳过 Jekyll 直接发布静态文件
└── themes/arknights/      # 主题（注意：是 submodule 引用，内容不入库，见下文）
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

```bash
# 命令行生成（自动套用 scaffolds/post.md 模板）
npx hexo new "文章标题"

# 或直接在 source/_posts/ 下手动创建 .md 文件
```

文章 front matter 格式：

```markdown
---
title: 文章标题
date: 2026-08-17 23:56:59
tags: [标签1, 标签2]
categories: [分类]
---

第一段内容…

<!-- more -->   <!-- 之前的部分作为首页摘要 -->

## 正文小节
```

### 修改 / 删除文章

- 修改：直接编辑 `source/_posts/文章名.md`
- 删除：删除对应文件即可

### 文章配图

图片放在 `source/images/` 下，文章中用相对路径引用（`_posts` 里的文件需回退一级）：

```markdown
![说明](../images/随机器/示例1.png)
```

### 发布上线（每次改完文章执行）

```bash
npm run clean      # 1. 清理旧生成文件
npm run build      # 2. 生成静态站点（自动把 .nojekyll 复制到 public）
npm run deploy     # 3. 推送到 gh-pages，GitHub 自动构建部署
git add -A         # 4. 备份源码到 source 分支
git commit -m "更新说明"
git push origin source
```

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

1. 先看 [GitHub 状态页](https://www.githubstatus.com) —— 若 Pages 处于故障/降级（429/503），等恢复后在 Actions 页面点 **Re-run jobs**，或重新 `npm run deploy`
2. 若 `npm run deploy` 提示 `nothing to commit`（内容没变化），用空提交强制触发构建：

```bash
cd .deploy_git
git commit --allow-empty -m "trigger build"
git push https://github.com/xingzhiyou/xingzhiyou.github.io.git HEAD:gh-pages
```

### 2. RSS 没推送

- 不是故障：阅读器有轮询周期，手动刷新即可
- 确认订阅的是 `https://ark.bd4wxr.top/atom.xml`

### 3. 换电脑后主题为空（重要）

`themes/arknights` 在 git 中是 **submodule 引用**（指向 [Yue-plus/hexo-theme-arknights](https://github.com/Yue-plus/hexo-theme-arknights) 的 commit `a876421`），但仓库里 **`.gitmodules` 缺失**，因此 clone 后主题目录是空的，会导致构建出空页面（`No layout` 警告、页面 0 字节）。

恢复方法：把该 commit 的主题内容放入 `themes/arknights/`：

```bash
git clone https://github.com/Yue-plus/hexo-theme-arknights.git /tmp/arknights-theme
cd /tmp/arknights-theme
git checkout a876421fc805edcd2ce961f760ca352147da4cc7
rsync -a --exclude='.git' ./ <项目路径>/themes/arknights/
```

**建议**：补回 `.gitmodules` 恢复 submodule，或把主题转为普通目录提交，让主题随仓库一起管理（可联系维护者协助）。

### 4. 让某篇文章重新"推送"一次给订阅者

修改该文章的 `date` 字段再重新发布即可（注意会同时改变归档/首页排序，非测试用途不建议频繁改）。

## npm 脚本

| 命令 | 作用 |
|---|---|
| `npm run clean` | `hexo clean`，清理 public 和数据库 |
| `npm run build` | `hexo generate && cp source/.nojekyll public/.nojekyll` |
| `npm run deploy` | `hexo deploy`，推送到 gh-pages 分支 |
| `npm run server` | `hexo server`，本地预览 |
