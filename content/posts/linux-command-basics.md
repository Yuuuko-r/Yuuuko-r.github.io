---
title: "Linux 命令行入门：目录导航与文件管理"
date: 2026-10-12
draft: false
categories: ["Linux 基础"]
tags: ["命令行", "文件管理", "Ubuntu"]
summary: "从 pwd、cd、ls 到 cp、mv、rm、find，把最常用的文件与目录操作命令整理成一份可随时查阅的笔记。"
---

刚装好 Ubuntu 的那几天，我最不习惯的就是没有"我的电脑"和资源管理器。后来发现，只要记住十几条命令，文件操作反而比图形界面更快。

## 一、我在哪：pwd、ls、cd

Linux 的目录是一棵树，`/` 是根。登录后所在的位置叫"当前工作目录"。

```bash
rsj@node1:~$ pwd
/home/rsj
rsj@node1:~$ ls -l
total 12
drwxr-xr-x 2 rsj rsj 4096 Oct 12 09:03 Desktop
drwxr-xr-x 2 rsj rsj 4096 Oct 12 09:03 Documents
-rw-r--r-- 1 rsj rsj  220 Oct 12 09:03 hello.txt
```

`ls` 的常用参数我记了四个：

| 参数 | 作用 |
| --- | --- |
| `-l` | 长格式，显示权限、属主、大小、时间 |
| `-a` | 显示隐藏文件（以 `.` 开头） |
| `-h` | 大小人类可读，如 `4.0K` |
| `-R` | 递归列出子目录 |

`cd` 的几个简写非常实用：`cd` 回到用户主目录，`cd ..` 上一级，`cd -` 回到上一次所在的目录。

## 二、路径的两种写法

- **绝对路径**：从根开始，如 `/etc/passwd`。
- **相对路径**：从当前位置开始，如 `./docs`、`../bin`。

其中 `.` 表示当前目录，`..` 表示上一级目录，`~` 表示当前用户的主目录（对 `rsj` 来说就是 `/home/rsj`）。

## 三、增删改查：mkdir、touch、cp、mv、rm

```bash
mkdir -p ~/linux-lab/src          # 一次创建多级目录
touch ~/linux-lab/readme.md       # 创建空文件
cp readme.md readme.bak           # 复制文件
cp -r src src_bak                 # 复制目录要加 -r
mv readme.bak doc/                # 移动；同一目录内移动就是重命名
rm -i doc/readme.bak              # 删除前确认
```

这里有个踩过的坑：`rm -rf` 没有回收站，删了就真没了。后来我养成了两个习惯——不确定时先 `ls` 看一眼通配符会匹配到哪些文件，以及永远不手输 `rm -rf /` 之类的危险命令。

## 四、通配符：一次操作一批文件

`*` 匹配任意多个字符，`?` 匹配单个字符，`[abc]` 匹配方括号中的任意一个字符。

```bash
ls *.txt            # 所有 txt 文件
ls file?.log        # file1.log、file2.log，不匹配 file10.log
ls [ab]*.sh         # 以 a 或 b 开头的 sh 脚本
```

## 五、找文件：find 与 locate

`find` 是"实时遍历"，`locate` 查的是数据库（需要 `updatedb` 更新），前者更准，后者更快。

```bash
find ~ -name "*.md"                       # 在主目录下按名字找
find / -name passwd 2>/dev/null           # 全盘查找，把无权限的报错丢掉
find ~ -name "*.log" -exec rm {} \;       # 找到后逐个执行命令
```

第二条里的 `2>/dev/null` 把标准错误重定向到"黑洞设备"，屏幕上就只剩下真正找到的结果了。

## 六、硬链接与软链接

```bash
ln hello.txt hello_hard      # 硬链接：同一个 inode，两个名字
ln -s hello.txt hello_soft   # 软链接：一个指向路径的"快捷方式"
```

- 硬链接不能跨文件系统，也不能给目录创建；删除源文件后内容仍可通过硬链接访问。
- 软链接可以跨文件系统、可以指向目录；源文件删除后软链接变成"断链"。

用 `ls -li` 看第一列的 inode 号就能验证：硬链接的 inode 与源文件相同，软链接的不同。

## 七、小结

| 任务 | 命令 |
| --- | --- |
| 看当前目录 | `pwd` |
| 列目录 | `ls -alh` |
| 切换目录 | `cd 路径` |
| 建目录 | `mkdir -p` |
| 复制 / 移动 | `cp -r` / `mv` |
| 删除 | `rm -i` |
| 查找 | `find 路径 -name 模式` |
| 链接 | `ln` / `ln -s` |

命令多不可怕，可怕的是不敢敲。建议在自己的家目录里随便建几个目录、拷几个文件练手，错了再删掉重来，成本几乎为零。
