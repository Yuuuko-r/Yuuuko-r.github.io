---
title: "文本处理三剑客：grep、sed、awk 与 Vim 生存指南"
date: 2026-11-16
draft: false
categories: ["文本处理"]
tags: ["vim", "grep", "sed", "awk", "管道"]
summary: "grep 找行、sed 改行、awk 按列加工，再配上管道和重定向，就能在命令行里完成大部分文本处理工作；文末附一份 Vim 常用操作速查。"
---

Linux 里几乎所有配置都是纯文本，所以"读文本、找文本、改文本"是绕不开的技能。这篇文章记录我对三剑客的理解和一个实际的统计数据练习。

## 一、grep：按行过滤

```bash
grep "root" /etc/passwd            # 包含 root 的行
grep -v "root" /etc/passwd         # 不包含 root 的行（-v 取反）
grep -n "bin" /etc/passwd          # 显示行号
grep -i "error" app.log            # 忽略大小写
grep -r "TODO" ~/code              # 递归搜索目录
grep -c "bin" /etc/passwd          # 只输出匹配的行数
```

`grep` 默认支持基础正则，加 `-E` 可以用扩展正则（等价于 `egrep`），`-o` 只输出匹配到的片段，这在提取字段时特别好用。

## 二、sed：按行编辑

`sed` 是流编辑器，按行读取、按规则修改、再输出，默认不改原文件。

```bash
sed 's/Linux/linux/' file.txt        # 每行只替换第一个
sed 's/Linux/linux/g' file.txt       # 全局替换
sed -i 's/8080/9090/g' config.conf   # 直接修改文件（-i）
sed -n '5,10p' file.txt              # 只打印第 5-10 行
sed '/^#/d' file.txt                 # 删除以 # 开头的注释行
```

`-i` 要慎用，改之前最好先不加 `-i` 跑一遍看看输出对不对。

## 三、awk：按列加工

`awk` 天生按"记录（行）—字段（列）"处理文本，默认以空白分隔：

```bash
awk '{print $1, $3}' access.log          # 打印第 1、3 列
awk -F: '{print $1}' /etc/passwd         # 指定冒号为分隔符
awk '$3 > 1000 {print $1}' /etc/passwd   # 条件过滤
awk '{sum += $5} END {print sum}' data   # 累加第 5 列
```

内置变量：`NR` 行号、`NF` 当前行字段数、`$0` 整行。`BEGIN{}` 在处理前执行，`END{}` 在处理后执行。

## 四、把三个工具串起来

作业里有一个统计练习：找出 `/etc/passwd` 中包含 `bin` 的行并计数。

```bash
cat /etc/passwd | grep bin | wc -l
```

更地道的写法是不用 `cat`：

```bash
grep -c bin /etc/passwd
```

再比如统计系统 CPU 物理个数：

```bash
grep 'physical id' /proc/cpuinfo | sort -u | wc -l
```

`sort -u` 先去重再计数，否则逻辑核会被重复统计。

## 五、管道与重定向

- `|` 把前一个命令的**标准输出**交给后一个命令的**标准输入**。
- `>` 覆盖写入文件，`>>` 追加，`<` 从文件读入。
- `2>` 重定向标准错误，`2>/dev/null` 丢弃错误，`&>` 同时重定向输出和错误。

三者的区别我记成一句话：**管道是"传下去"，重定向是"存起来"**。

## 六、Vim 生存指南

Vim 有三种模式：命令模式（默认）、插入模式（`i`/`a`/`o` 进入）、末行模式（`:` 进入）。按 `Esc` 回到命令模式。

| 操作 | 按键 |
| --- | --- |
| 保存退出 | `:wq` 或 `ZZ` |
| 不保存退出 | `:q!` |
| 显示行号 | `:set nu` |
| 跳到第 n 行 | `:n` 或 `ngg` |
| 行首 / 行尾 | `0` / `$` |
| 删除一行 | `dd` |
| 复制 / 粘贴 | `yy` / `p` |
| 撤销 | `u` |
| 向下查找 | `/关键词`，`n` 下一个 |
| 替换全部 | `:%s/old/new/g` |

我刚开始最常犯的错是：忘了按 `Esc` 就直接敲 `:wq`，结果 `:` 被当成普通字符插进了正文。练几次形成肌肉记忆就好了。

## 七、小结

这三个工具各有分工：**grep 负责"挑"，sed 负责"改"，awk 负责"算"**。配合管道，很多原本要写脚本甚至写程序才能完成的事情，一行命令就够了。刚学的时候不必背参数，记住思路，用的时候 `man grep` 查一下即可。
