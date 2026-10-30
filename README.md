# RTMQ 实验控制文档

记录 TIQC 实验中 RTMQ、Seq 和 Device 的安装、配置、使用方法与调试经验。本仓库维护的是文档；实验控制软件和设备驱动需要按文档中的说明另行获取。

- [使用文档](https://xiaoshecode.github.io/rtmq/)
- [仓库维护与检查记录](https://xiaoshecode.github.io/xiaoshecode/repos/rtmq/)

## 本地预览与检查

使用 Python 3.10 或更新版本，在虚拟环境中安装文档依赖：

```sh
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS / Linux: source .venv/bin/activate
python -m pip install -r requirements-docs.txt
python -m mkdocs serve
python -m mkdocs build --strict
```

预览地址由 `mkdocs serve` 输出。构建产物位于 `site/`，无需提交到源码分支。

## 目录与维护规范

| 路径 | 用途 |
| --- | --- |
| `docs/index.md` | 使用文档首页 |
| `docs/Seq*.md` | Seq 安装、配置与使用 |
| `docs/Device*.md` | Device 安装、配置与使用 |
| `docs/DebugLog.md` | 实验调试记录 |
| `docs/figure/` | 文档插图 |
| `mkdocs.yml` | 导航、主题和插件配置 |
| `requirements-docs.txt` | 本地文档构建依赖 |
| `.github/workflows/ci.yml` | GitHub Pages 部署 |

新增页面时同步维护 `mkdocs.yml` 导航，使用相对 Markdown 链接。提交前运行严格构建；涉及设备的操作须在对应实验台验证后记录环境、日期及结果。文档构建成功不代表硬件操作已验证。

## 发布

现有 GitHub Actions 工作流在 `main` / `master` 更新后部署到 `gh-pages`，网站地址保持不变。GitLab 镜像使用 `.gitlab-ci.yml` 构建 `public/`。
