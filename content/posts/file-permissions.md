---
title: "文件权限与用户管理：rwx 到底怎么算"
date: 2026-10-26
draft: false
categories: ["Linux 基础"]
tags: ["权限", "chmod", "用户管理"]
summary: "用 ls -l 的九位权限串拆解 rwx，讲清 chmod 的数字法与符号法、属主属组、umask，以及 useradd、usermod 等用户管理命令。"
---

第一次看到 `-rw-r--r--` 这种输出时我完全没概念。这次的作业是把它们彻底搞明白，并且亲手验证了一遍"去掉读权限之后 vim 打不开文件"。

## 一、ls -l 的每一段是什么意思

```bash
rsj@node1:~/testchd$ ls -l a.txt
-rw-r--r-- 1 rsj rsj 32 Oct 26 10:21 a.txt
```

从左到右依次是：

1. **文件类型**：`-` 普通文件、`d` 目录、`l` 软链接、`c` 字符设备、`b` 块设备。
2. **权限**：共 9 位，每 3 位一组，分别对应**属主（u）**、**属组（g）**、**其他人（o）**，每组是 `r`（读）、`w`（写）、`x`（执行）的有无。
3. **硬链接数**。
4. **属主**：这里是 `rsj`。
5. **属组**：这里也是 `rsj`。
6. **大小**（字节）。
7. **修改时间**。
8. **文件名**。

上面这行 `-rw-r--r--` 的意思就是：属主可读可写，属组和其他人只可读。

## 二、chmod：两种写法

**数字法**：`r=4`、`w=2`、`x=1`，三位相加。

```bash
chmod 755 a.txt    # rwxr-xr-x
chmod 644 a.txt    # rw-r--r--
chmod 600 a.txt    # rw-------
```

**符号法**：`u`/`g`/`o`/`a` 加 `+`/`-`/`=`。

```bash
chmod u+x a.txt    # 属主增加执行权限
chmod go-r a.txt   # 属组和其他人去掉读权限
chmod a=r a.txt    # 所有人只保留读权限
```

## 三、一次课上验证：脚本为什么执行不了

我按作业要求在 `~/testchd` 下写了一个脚本 `a.txt`：

```bash
#!/bin/bash
echo I love chd
```

直接运行 `./a.txt`，系统报 `Permission denied`。原因是新建文件的默认权限是 `644`，没有执行位。加上执行权限后立即就正常了：

```bash
chmod u+x a.txt
ls -l a.txt
-rwxr--r-- 1 rsj rsj 32 Oct 26 10:26 a.txt
./a.txt
I love chd
```

> 注意：`./a.txt` 里的 `./` 表示"当前目录下的这个文件"。如果只输入 `a.txt`，Shell 只会去 `PATH` 里的目录查找，找不到就会报 `command not found`。

## 四、把读权限去掉会怎样

继续做实验：`chmod u-r a.txt` 之后用 `vim a.txt` 打开，vim 会在状态栏提示文件不可读，只能以只读方式查看部分内容甚至直接报错；对普通用户而言，缺少读权限时连 `cat` 都会失败。

这说明：**权限是内核对文件访问的强制检查，不是编辑器的限制**。图形界面、命令行、脚本，谁都无法绕过。

## 五、umask 与默认权限

新建普通文件的默认权限是 `666 & ~umask`，目录是 `777 & ~umask`。Ubuntu 默认 `umask` 为 `002`，所以新文件是 `664`、新目录是 `775`。用 `umask` 命令可以查看或临时修改。

## 六、属主与属组：chown、chgrp

```bash
sudo chown root a.txt          # 只改属主
sudo chown root:root a.txt     # 同时改属主和属组
sudo chgrp www-data a.txt      # 只改属组
chown -R rsj:rsj ~/testchd     # 递归修改整个目录
```

普通用户只能修改自己文件的属组（且必须是自己的组），改属主必须用 `sudo`。

## 七、用户管理常用命令

| 命令 | 作用 |
| --- | --- |
| `useradd -m -s /bin/bash tom` | 新建用户并创建家目录、指定 shell |
| `passwd tom` | 设置密码 |
| `usermod -aG sudo tom` | 追加到 sudo 组（`-a` 不能省，否则会覆盖原有组） |
| `userdel -r tom` | 删除用户并删除家目录 |
| `groups tom` | 查看用户所属组 |
| `id` | 查看当前用户的 uid、gid 和组 |

超级用户 `root` 的 uid 为 0，家目录是 `/root`。用 `id -u` 等于 0、或者提示符末尾是 `#` 都能判断当前是不是 root。

## 八、小结

权限这部分看起来枯燥，但它是 Linux 安全模型的地基：Web 服务读不到配置就起不来、脚本没有 `x` 就跑不动、密钥文件必须是 `600`……把 `rwx` 和 `chmod` 弄懂之后，很多"莫名其妙"的问题就都有解释了。
