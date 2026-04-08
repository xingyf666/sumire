# Sumire 个人主页

使用 [MkDocs](https://www.mkdocs.org/) + [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) 构建。

## 🏃 快速开始

```bash
# 安装依赖
pip install mkdocs-material

# 本地预览
mkdocs serve

# 构建静态文件
mkdocs build
```

## 📁 项目结构

```
my-site/
├── docs/              # Markdown 文档
│   ├── index.md       # 主页
│   ├── about/         # 关于页面
│   ├── blog/          # 博客文章
│   └── projects/      # 项目展示
├── .github/
│   └── workflows/     # GitHub Actions
├── mkdocs.yml         # 配置文件
└── README.md
```

## 🚀 部署到 GitHub Pages

1. 创建 GitHub 仓库，将项目推送上去
2. 进入仓库 → Settings → Pages
3. Source 选择 **GitHub Actions**
4. 推送代码到 main 分支，自动部署

## 🎨 自定义

编辑 `mkdocs.yml` 修改：
- 站点名称、描述
- 导航结构
- 主题颜色
- 功能特性

---

*Powered by MkDocs & Material for MkDocs*
