# 任思洁的个人博客

基于 **Hugo + PaperMod** 搭建的个人博客，课程大作业。同一份源码支持两种访问方式：

- **本地访问**：在 Linux 虚拟机中 `hugo server --bind 0.0.0.0 -D`，宿主机浏览器访问 `http://<虚拟机IP>:1313`
- **公网访问**：推送到 GitHub，由 GitHub Actions 自动构建并发布到 GitHub Pages

## 目录结构

```
blog/
├── hugo.toml                   # 站点配置（主题、菜单、首页信息、输出格式）
├── content/
│   ├── about/index.md          # 关于页（含个人照片）
│   ├── search.md               # 搜索页（layout: search）
│   ├── archives.md             # 归档页（layout: archives）
│   └── posts/                  # 文章
├── themes/PaperMod             # 主题（git submodule）
├── .github/workflows/hugo.yaml # GitHub Actions 自动部署
└── .gitignore                  # 忽略 public/ 等构建产物
```

## 本地运行

```bash
# 克隆时记得拉取子模块
git clone --recursive https://github.com/Yuuuko-r/blog.git
cd blog

# 预览（草稿也渲染）
hugo server --bind 0.0.0.0 -D
```

浏览器打开 `http://<虚拟机IP>:1313`。

## 写新文章

```bash
hugo new posts/my-new-post.md
```

编辑 `content/posts/my-new-post.md`，把 front matter 里的 `draft` 改为 `false` 即可出现在首页。

## 部署

推送到 `main` 分支后，GitHub Actions 自动完成构建与发布。仓库 **Settings → Pages → Source** 需选择 **GitHub Actions**。

## 遇到的问题与解决办法

| 问题 | 现象 | 解决办法 |
| --- | --- | --- |
| Hugo 版本过旧 | `apt install hugo` 装到 0.68，PaperMod 报 SCSS 相关错误 | 下载官方 `hugo_extended` 二进制安装到 `/usr/local/bin` |
| 宿主机访问不到 1313 | 宿主机浏览器打不开 `http://IP:1313` | `hugo server` 加 `--bind 0.0.0.0`；必要时 `sudo ufw allow 1313/tcp` |
| 主题目录为空 | Actions 构建报 `found no layout file` | checkout 步骤加 `submodules: recursive` |
| 子路径样式丢失 | Pages 页面无 CSS | `baseURL` 写成 `https://Yuuuko-r.github.io/blog/`，或用 `--baseURL` 覆盖 |
| 搜索无结果 | 搜索页能打开但搜不到内容 | 确认 `[outputs] home = ["HTML", "RSS", "JSON"]` 且 `content/search.md` 的 `layout: "search"` |
| `github.com` 解析异常 | `git clone` 超时 | 在 `/etc/hosts` 中固定可用 IP，或用镜像加速下载 |

## 许可

文章内容原创，主题版权归 [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 作者所有。
