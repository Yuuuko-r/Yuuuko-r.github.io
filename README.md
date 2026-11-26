# 任思洁的个人博客

基于 **Hugo + PaperMod** 搭建的个人博客，课程大作业。同一份源码支持两种访问方式：

- **方式 A 本地访问**：在 Linux 虚拟机中 `hugo server --bind 0.0.0.0 -D`，宿主机浏览器访问 `http://<虚拟机IP>:1313`
- **方式 B 公网访问**：推送到 GitHub，由 GitHub Actions 自动构建并发布到 GitHub Pages（`https://Yuuuko-r.github.io/`）

## 目录结构

```
.
├── hugo.toml                   # 站点配置（主题、六项菜单、首页信息、输出格式）
├── content/
│   ├── about/index.md          # 关于页（姓名、专业、个人照片）
│   ├── search.md               # 搜索页（layout: search）
│   ├── archives.md             # 归档页（layout: archives）
│   └── posts/                  # 文章（draft: false）
├── themes/PaperMod             # 主题（git submodule）
├── .github/workflows/hugo.yaml # GitHub Actions 自动部署
└── .gitignore                  # 忽略 public/ 等构建产物
```

## 环境版本

| 组件 | 版本 | 说明 |
| --- | --- | --- |
| 操作系统 | Ubuntu 20.04.6 LTS（glibc 2.31） | 虚拟机 node1 |
| Hugo | v0.145.0 extended | 官方二进制，安装到 `/usr/local/bin/hugo` |
| PaperMod | commit 149f12c（2025-04-14） | 与该 Hugo 版本兼容的最后一个提交 |
| Git | 2.25.1 | 用于子模块与推送 |

## 本地运行

```bash
# 克隆时记得拉取子模块（主题在 themes/PaperMod）
git clone --recursive https://github.com/Yuuuko-r/Yuuuko-r.github.io.git
cd Yuuuko-r.github.io

# 预览（-D 表示草稿也渲染）
hugo server --bind 0.0.0.0 -D
```

浏览器打开 `http://<虚拟机IP>:1313`（本机为 `http://192.168.56.101:1313`）。

## 写新文章

```bash
hugo new posts/my-new-post.md
```

编辑 `content/posts/my-new-post.md`，把 front matter 里的 `draft` 改为 `false` 即可出现在首页。

## 部署

推送到 `main` 分支后，GitHub Actions 自动完成构建与发布。仓库 **Settings → Pages → Source** 需选择 **GitHub Actions**。
工作流使用 `actions/checkout`（`submodules: recursive`）、`peaceiris/actions-hugo`（Hugo extended）与 `actions/deploy-pages`。

## 遇到的问题与解决办法

| 问题 | 现象 | 解决办法 |
| --- | --- | --- |
| Hugo 版本不兼容 | 最新 0.158.0 二进制在 20.04 上无法启动：`libc.so.6: version 'GLIBC_2.33' not found` | 改用与 glibc 2.31 兼容的 **v0.145.0 extended** |
| 主题版本不兼容 | PaperMod master 要求 Hugo ≥ 0.158；`v8.0` 使用已移除的 `.Site.Social` 报错 | 检出 `149f12c`（2025-04-14，0.146 模板改造之前的最后一个提交） |
| 宿主机访问不到 1313 | 宿主机浏览器打不开 `http://IP:1313` | `hugo server` 加 `--bind 0.0.0.0`；必要时 `sudo ufw allow 1313/tcp` |
| 主题目录为空 | Actions 构建报 `found no layout file` | checkout 步骤加 `submodules: recursive` |
| 页面样式丢失 | Pages 页面无 CSS | 用户站点 `baseURL` 固定为 `https://Yuuuko-r.github.io/`，或用 `--baseURL` 覆盖 |
| 搜索无结果 | 搜索页能打开但搜不到内容 | 确认 `[outputs] home = ["HTML", "RSS", "JSON"]` 且 `content/search.md` 的 `layout: "search"` |
| `github.com` 解析异常 | `git clone`/`push` 超时（DNS 被污染，443 被拒） | `/etc/hosts` 固定可用 IP；改用 `ssh.github.com:443` 推送；主题克隆走 `ghfast.top` 镜像 |
| 图片不显示 | 关于页头像 404 | 图片放在 `content/about/` 下并在 Markdown 中使用相对路径 |

## 许可

文章内容原创，主题版权归 [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 作者所有。
