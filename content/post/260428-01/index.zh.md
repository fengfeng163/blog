---
title: 在github上建知识库的方案
date: 2026-04-28
description: 最近被语雀知识库的每月100篇限制卡住了。问了下Claude 4.7, 发现原来在github上建知识库，原来有这么多种优秀而且持有免费的方案。
tags: 
    - 效率工具
    - 知识库
categories:
    - 自我提升
---
# 在 GitHub 上建知识库完全可行

GitHub 不仅可以建知识库,而且是目前**最流行的个人/团队知识库方案之一**。很多知名的知识库(如各种 Awesome 系列、技术博客、学习笔记)都托管在 GitHub 上。

## 为什么用 GitHub 建知识库

| 优势 | 说明 |
|------|------|
| **免费** | 公开/私有仓库都免费,空间足够个人使用 |
| **版本管理** | 每次修改都有历史记录,可回溯、可对比 |
| **多端同步** | 通过 git 在多台电脑同步,不依赖单一设备 |
| **Markdown 友好** | GitHub 原生渲染 Markdown,显示效果好 |
| **可发布为网站** | 通过 GitHub Pages 一键发布成在线网站 |
| **协作便利** | 支持多人编辑、PR、Issue 讨论 |
| **搜索强大** | 支持全文搜索、代码搜索 |
| **生态丰富** | 配套工具(Obsidian、VSCode、各种静态站点生成器) |

## 主流方案对比

### 方案 1: 纯 Markdown 仓库(最简单)

直接在 GitHub 仓库里放 `.md` 文件,通过文件夹组织目录结构。

```
my-knowledge-base/
├── README.md              # 知识库首页/索引
├── 编程语言/
│   ├── Python基础.md
│   ├── JavaScript笔记.md
│   └── Go语言入门.md
├── 数据库/
│   ├── PostgreSQL命令.md
│   ├── MySQL优化.md
│   └── Redis实战.md
├── 工具/
│   ├── Git使用.md
│   └── Docker入门.md
└── 读书笔记/
    └── 设计模式.md
```

**优点**: 零配置、零成本、即开即用
**缺点**: 没有漂亮的网站界面、搜索体验一般

适合: 入门、纯个人笔记、不追求美观

### 方案 2: GitHub Pages + 静态站点生成器(推荐)

把 Markdown 自动渲染成漂亮的网站,免费部署在 `https://用户名.github.io/仓库名`。

#### 主流静态站点工具

| 工具 | 特点 | 适合人群 |
|------|------|---------|
| **Docsify** | 无需构建、即写即用、配置最简单 | 新手、想快速上线 |
| **VuePress** | 基于 Vue,主题丰富 | 前端开发者 |
| **VitePress** | VuePress 的升级版,极快 | 前端开发者 |
| **MkDocs** | Python 生态,Material 主题非常好看 | Python 用户、追求美观 |
| **Hugo** | 速度极快(Go 编写) | 大量文章、技术博客 |
| **Hexo** | Node.js 生态,中文社区活跃 | 写博客 |
| **Docusaurus** | Facebook 出品,功能强大 | 项目文档 |
| **GitBook** | 像书一样阅读体验 | 写电子书 |

### 方案 3: Wiki 功能(GitHub 自带)

每个 GitHub 仓库都自带 Wiki 功能,在仓库页面顶部的 `Wiki` 标签可以开启。

**优点**: 不用配置、有目录侧边栏、支持 Markdown
**缺点**: 不能本地同步管理(只能网页编辑或单独 clone wiki 仓库)、SEO 较差

适合: 项目文档、临时笔记

### 方案 4: GitHub + Obsidian(笔记软件)联动

这是目前**最强大的个人知识库方案**之一。

```
本地 Obsidian 仓库 ←→ Git 同步 ←→ GitHub 仓库
```

- **Obsidian**: 本地 Markdown 编辑器,支持双向链接、关系图谱、插件丰富
- **GitHub**: 作为云端备份和版本管理

**优点**: 本地编辑体验极佳 + 云端备份 + 多端同步
**缺点**: 需要学习 Obsidian、需要配置 git 同步插件

## 快速上手:5 分钟搭建一个知识库

### 最简方案(纯 Markdown 仓库)

```bash
# 1. 在 GitHub 创建一个新仓库,如 my-notes

# 2. 本地初始化
mkdir my-notes && cd my-notes
git init

# 3. 创建首页和分类
echo "# 我的知识库" > README.md
mkdir 数据库 编程 工具

# 4. 写笔记
echo "# PostgreSQL 命令" > 数据库/PostgreSQL.md

# 5. 提交到 GitHub
git add .
git commit -m "init knowledge base"
git branch -M master
git remote add origin git@github.com:用户名/my-notes.git
git push -u origin master
```

完成!访问 `https://github.com/用户名/my-notes` 即可看到你的知识库,GitHub 会自动渲染 Markdown。

### 进阶方案(Docsify - 推荐新手)

Docsify 不需要构建,直接把 Markdown 渲染成网站。

#### 步骤

```bash
# 1. 在仓库根目录创建 index.html
```

`index.html` 内容:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>我的知识库</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify@4/themes/vue.css">
</head>
<body>
  <div id="app">加载中...</div>
  <script>
    window.$docsify = {
      name: '我的知识库',
      repo: 'https://github.com/用户名/my-notes',
      loadSidebar: true,
      subMaxLevel: 3,
      search: 'auto'
    }
  </script>
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/search.min.js"></script>
</body>
</html>
```

```bash
# 2. 创建侧边栏 _sidebar.md
```

`_sidebar.md` 内容:

```markdown
- [首页](README.md)
- 数据库
  - [PostgreSQL](数据库/PostgreSQL.md)
  - [MySQL](数据库/MySQL.md)
- 编程
  - [Python](编程/Python.md)
- [关于](关于.md)
```

```bash
# 3. 推送到 GitHub
git add .
git commit -m "use docsify"
git push

# 4. 启用 GitHub Pages
# 仓库 Settings → Pages → Source 选择 master 分支 → 保存
# 访问 https://用户名.github.io/my-notes 即可
```

### 进阶方案(MkDocs Material - 最美观)

如果追求专业、美观的文档站,推荐 **MkDocs + Material 主题**。许多大厂的开源文档(如 FastAPI、Pydantic)都使用它。

```bash
# 1. 安装(需要 Python)
pip install mkdocs-material

# 2. 创建项目
mkdocs new my-notes
cd my-notes

# 3. 编辑 mkdocs.yml
```

`mkdocs.yml`:

```yaml
site_name: 我的知识库
theme:
  name: material
  language: zh
  features:
    - navigation.tabs
    - search.suggest
    - search.highlight
nav:
  - 首页: index.md
  - 数据库:
    - PostgreSQL: db/postgres.md
  - 编程:
    - Python: code/python.md
```

```bash
# 4. 本地预览
mkdocs serve
# 浏览器打开 http://127.0.0.1:8000

# 5. 部署到 GitHub Pages
mkdocs gh-deploy
```

完成后访问 `https://用户名.github.io/my-notes`,效果非常专业。

## 知识库目录结构建议

### 按主题分类(推荐)

```
knowledge-base/
├── README.md
├── 00-索引/
│   └── 全部文章索引.md
├── 01-编程语言/
│   ├── Python/
│   ├── JavaScript/
│   └── Go/
├── 02-数据库/
├── 03-框架/
├── 04-工具/
├── 05-架构/
├── 06-读书笔记/
├── 07-面试题/
└── 99-杂项/
```

### 按 PARA 法则(知识管理理论)

```
knowledge-base/
├── 1-Projects/      # 当前进行中的项目笔记
├── 2-Areas/         # 长期关注的领域
├── 3-Resources/     # 参考资料
└── 4-Archives/      # 已归档内容
```

## 实用建议

### 1. 公开还是私有

| 类型 | 适合内容 |
|------|---------|
| **公开仓库** | 技术文章、学习笔记、教程、可分享的内容 |
| **私有仓库** | 工作机密、个人日记、未完成的草稿 |

GitHub 私有仓库免费,可以放心用。

### 2. README.md 当目录页

把 `README.md` 作为知识库首页,列出所有分类的链接,方便快速跳转。

```markdown
# 我的知识库

## 📚 目录

- [数据库](./数据库/README.md)
  - [PostgreSQL 命令](./数据库/PostgreSQL.md)
  - [MySQL 优化](./数据库/MySQL.md)
- [编程语言](./编程/README.md)
  - [Python 笔记](./编程/Python.md)
- [工具](./工具/README.md)

## 🔥 最近更新

- 2026-04-28: 整理 PostgreSQL 常用命令
- 2026-04-25: Docker 入门指南
```

### 3. 文件命名规范

```
✅ 推荐
PostgreSQL常用命令.md
docker-compose入门.md
2026-04-28-学习笔记.md

❌ 避免
新建文档.md
笔记1.md
test.md
```

### 4. 善用图床

Markdown 中的图片建议放在仓库的 `assets/` 或 `images/` 目录里,使用相对路径:

```markdown
![架构图](./assets/architecture.png)
```

或使用 GitHub Issue 上传图片,自动得到一个图床链接。

### 5. 配合 VSCode 编辑

推荐扩展:
- **Markdown All in One**: 增强 Markdown 编辑
- **Markdown Preview Enhanced**: 实时预览
- **GitLens**: 增强 git 功能
- **Paste Image**: 粘贴图片自动保存到本地

## 推荐工具组合

### 个人知识库(推荐)

```
Obsidian(本地编辑) + GitHub(云端备份) + Obsidian Git 插件(自动同步)
```

体验最好,功能最强,支持双向链接和关系图谱。

### 技术博客/文档

```
VSCode(编辑) + Hugo / VitePress(生成) + GitHub Pages(部署)
```

适合写公开技术文章。

### 团队知识库

```
GitHub 私有仓库 + MkDocs / Docusaurus + GitHub Pages
```

适合团队协作、项目文档。

## 知名 GitHub 知识库参考

可以看看别人是怎么组织的(可在 GitHub 搜索):

- **CS-Notes**: 计算机基础知识
- **awesome-***: 各种领域精选资源
- **interview_internal_reference**: 面试题
- **JavaGuide**: Java 学习指南
- **mkdocs-material 官方文档**: MkDocs 范例

## 总结

**GitHub 建知识库不仅可行,而且非常推荐**。根据需求选择方案:

- **零基础、就想记笔记**: 纯 Markdown 仓库,5 分钟搞定
- **想要漂亮的网站**: Docsify(最简单)或 MkDocs Material(最美观)
- **追求最佳体验**: Obsidian + GitHub 双向同步
- **写技术博客**: Hugo / Hexo + GitHub Pages

最重要的不是工具,而是**坚持记录和整理**。先用最简单的方案开始,等积累一定内容后,再考虑升级到更复杂的方案。

## 最后：个人行动指南
### 语雀
以后的话，语雀就写隐私的东西。比如日记，自己的计划，等各种不应该公开的信息。
其实github的私有仓库也能实现同样的功能。不过，暂时使用语雀吧。
### Hugo技术博客
技术博客用hugo去写，用来包装自己的形象，练一下日语、英语。虽然也是托管在github上的。但是独立起来更好点吧。---公开，总的来说是给别人看的。
注意：现在我的仓库是公有的，后面这个时间改成私有的，不然别人很容易吧我的文件轻松搬家拿走。
### Github知识库
Github建一个知识库，整理，积累自己的知识体系。---公开，但主要是给自己看的。
obsidian + github + Quartz 最推荐。-->
### 产品介绍等
obsidian + github+ MkDocs Material最推荐。---公开