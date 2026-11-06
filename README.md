# HexYans's Blog

我的私人博客，基于 Homecraft（Minecraft 风格 3D 个人网站模板）。

- 线上：<https://blog.hexyans.com>
- 3D 世界：`world/`（房子、泳池、家具、后山等）
- 博客文章：`posts/`（每篇一个 HTML）

## 本地预览

```sh
python3 tools/serve.py
```

## 发布

推送到 `main` 后由 GitHub Pages 自动部署（`.github/workflows/pages.yml`）。
手动打包：`bash tools/build.sh`，产物在 `dist/`。

## 写文章

复制 `posts/_template.html` 为新文件，改标题/日期/正文，再到 `index.html` 的
`data-station="blog"` 卡片里加一条链接。

## 目录

| 路径 | 作用 |
|---|---|
| `index.html` | 卡片内容（关于/博客/作品/联系/留言） |
| `home.config.js` | 站点设置（名字、皮肤、语言、Dock、留言、传送门） |
| `homecraft.json` | 对外公开元数据（门户/目录读取） |
| `world/` | 3D 世界（`layout.js` 数据，`build.js` 搭建） |
| `posts/` | 博客文章 |
| `assets/` | 图片、皮肤、图标 |
| `engine/` | 共享引擎（请勿修改） |

站点内容与素材均可自行更换。基于 Homecraft，遵循 MIT 许可（见 `LICENSE`）。
