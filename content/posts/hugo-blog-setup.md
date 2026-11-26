---
title: "在 Linux 虚拟机上用 Hugo + PaperMod 搭建个人博客"
date: 2026-11-23
draft: false
categories: ["建站"]
tags: ["Hugo", "PaperMod", "GitHub Pages", "Ubuntu"]
summary: "从零开始在 Ubuntu 虚拟机里安装 Hugo、用 git submodule 引入 PaperMod、配置首页与六大菜单，再用 GitHub Actions 自动发布到 GitHub Pages 的完整过程。"
---

这个博客本身就是这门课的期末大作业。从装 Hugo 到自动部署上线，前后折腾了大概一周，踩的坑比想象中多，所以把过程完整记下来。

## 一、环境

- 宿主机：Windows 10，VirtualBox 7
- 虚拟机：Ubuntu 20.04.6 LTS，主机名 `node1`，IP `192.168.56.101`（host-only 网卡）
- Hugo：v0.145.0 extended
- 主题：[PaperMod](https://github.com/adityatelange/hugo-PaperMod)

## 二、安装 Hugo

Ubuntu 自带源里的 Hugo 版本太老（0.68），PaperMod 用不了，所以直接下官方二进制：

```bash
wget https://github.com/gohugoio/hugo/releases/download/v0.145.0/hugo_extended_0.145.0_linux-amd64.tar.gz
tar xzf hugo_extended_0.145.0_linux-amd64.tar.gz
sudo install -m 755 hugo /usr/local/bin/hugo
hugo version
```

安装 extended 版本是因为主题编译 SCSS 时需要它，普通版本会报 `this feature is not available in your current Hugo version`。

## 三、创建站点并引入主题

```bash
hugo new site ~/blog --format toml
cd ~/blog
git init
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

用 `git submodule` 而不是直接 `git clone` 的好处是：主题版本能跟着主仓库一起被记录，克隆仓库时加 `--recursive` 就能把主题一起拉下来。

常用检查命令：

```bash
cat .gitmodules
git submodule status
ls themes/PaperMod
```

## 四、hugo.toml 的三个关键配置

**1. 指定主题**

```toml
theme = "PaperMod"
```

**2. 让首页输出 JSON**

```toml
[outputs]
  home = ["HTML", "RSS", "JSON"]
```

PaperMod 的搜索功能依赖站点根目录下的 `index.json`。没有这一行，搜索页能打开但搜不到任何结果。

**3. 六大菜单项**

```toml
[menu]
  [[menu.main]]
    identifier = "home"
    name = "首页"
    url = "/"
    weight = 10
  # 关于 /about/、标签 /tags/、分类 /categories/、归档 /archives/、搜索 /search/ 同理
```

`weight` 决定菜单顺序，数字小的排在前面。归档页和搜索页需要自己建 `content/archives.md` 和 `content/search.md`，并在 front matter 里指定布局：

```yaml
# content/search.md
---
title: "搜索"
layout: "search"
---
```

```yaml
# content/archives.md
---
title: "归档"
layout: "archives"
---
```

## 五、本地预览

```bash
hugo server --bind 0.0.0.0 -D
```

默认只监听 `127.0.0.1`，虚拟机里的浏览器能访问，但宿主机访问不了。加上 `--bind 0.0.0.0` 后监听所有网卡，宿主机浏览器打开 `http://192.168.56.101:1313` 就能看到站点。`-D` 表示把草稿也渲染出来，方便边写边看。

如果宿主机还是打不开，先检查虚拟机的防火墙：

```bash
sudo ufw allow 1313/tcp
sudo ufw status
```

## 六、用 GitHub Actions 自动部署

在仓库里新建 `.github/workflows/hugo.yaml`，核心步骤是：

1. `actions/checkout`，**必须带 `submodules: recursive`**，否则主题目录是空的，构建直接失败；
2. `peaceiris/actions-hugo` 安装指定版本的 Hugo（extended）；
3. `hugo --minify` 生成 `public/`；
4. 上传 artifact 并用 `actions/deploy-pages` 发布。

然后到仓库 **Settings → Pages**，把 Source 选成 **GitHub Actions**。之后每次 `git push`，Actions 都会自动构建并把站点发布到 `https://用户名.github.io/仓库名/`。

## 七、踩过的坑

**1. `github.com` 解析异常**
虚拟机上 `getent hosts github.com` 返回了一个奇怪的 IP，`git clone` 一直超时。后来在 `/etc/hosts` 里手动指定了可用的 GitHub 地址才恢复正常。临时方案还可以用镜像加速下载。

**2. 主题目录是空的**
第一次部署 Actions 报错 `found no layout file`。原因是 checkout 时没拉取子模块——加上 `submodules: recursive` 解决。

**3. 子路径下样式 404**
项目型 Pages 的地址是 `https://用户名.github.io/仓库名/`，如果 `baseURL` 写成了根域名，CSS 会全部 404。把 `baseURL` 写成带仓库名的完整路径，或者在 workflow 里用 `--baseURL` 参数覆盖即可。

**4. 搜索没结果**
就是前面说的 `[outputs] home` 少了 `JSON`，或者 `search.md` 的 `layout` 写错。

## 八、总结

Hugo 的构建速度非常快（这个站点几十篇文章也是毫秒级），PaperMod 的默认样式简洁，几乎不用改模板。真正花时间的部分都在环境：网络、子模块、Pages 的路径配置。把这几处理顺之后，写文章就只是 `hugo new posts/xxx.md` 然后提交了。
