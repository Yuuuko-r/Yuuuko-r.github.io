---
title: "Shell 脚本实战：从 one2n 求和到猜年龄游戏"
date: 2026-11-09
draft: false
categories: ["Shell 编程"]
tags: ["bash", "脚本", "流程控制"]
summary: "用两个小脚本串起变量、参数判断、if/case、for/while 循环和函数：one2n 求和与 age_room 猜数字，附上执行方式差异的解释。"
---

Shell 脚本最吸引我的地方是：几十行代码就能把一件重复的事自动化。这篇笔记记录我写的两个练习脚本，以及踩过的几个坑。

## 一、脚本的第一行和第一步

```bash
#!/bin/bash
```

这叫 shebang，告诉系统用哪个解释器执行这个文件。写完记得加执行权限：

```bash
chmod +x one2n
./one2n 10
```

如果直接 `bash one2n`，其实不需要执行权限，因为文件是被 bash 当作参数读进去的。

## 二、练习一：one2n 求和

需求：接收一个大于 0 的整数 n，计算 1+2+…+n；不传参数时打印用法。

```bash
#!/bin/bash
# one2n: 计算 1 到 n 的和

if [ $# -ne 1 ]; then
    echo "用法: $0 <正整数>"
    exit 1
fi

n=$1
if ! [[ "$n" =~ ^[0-9]+$ ]] || [ "$n" -le 0 ]; then
    echo "错误: 参数必须是大于 0 的整数"
    exit 1
fi

sum=0
for ((i = 1; i <= n; i++)); do
    sum=$((sum + i))
done

echo "1+2+...+$n = $sum"
```

运行效果：

```bash
rsj@node1:~$ ./one2n 100
1+2+...+100 = 5050
rsj@node1:~$ ./one2n
用法: ./one2n <正整数>
rsj@node1:~$ ./one2n abc
错误: 参数必须是大于 0 的整数
```

这里用到的知识点：

- `$#` 是参数个数，`$0` 是脚本名，`$1` 是第一个参数。
- `[ ... ]` 实际上是 `test` 命令的简写，`-ne` 是不等于。
- `[[ ... ]]` 是 bash 的扩展语法，支持 `=~` 正则匹配。
- `$(( ))` 做整数运算，`$((sum + i))` 不需要写 `$sum`。

## 三、练习二：age_room 猜数字

需求：脚本随机生成 1–100 的年龄，用户循环猜，提示"大了/小了"，猜中后显示次数。

```bash
#!/bin/bash
# age_room.sh: 猜年龄小游戏

target=$((RANDOM % 100 + 1))
count=0

echo "我想好了一个 1-100 之间的年龄，你来猜猜看！"

while true; do
    read -p "请输入你的猜测: " guess
    count=$((count + 1))

    if ! [[ "$guess" =~ ^[0-9]+$ ]]; then
        echo "请输入数字～"
        continue
    fi

    if [ "$guess" -eq "$target" ]; then
        echo "恭喜你，猜对了！一共猜了 $count 次。"
        break
    elif [ "$guess" -lt "$target" ]; then
        echo "小了一点，再试试。"
    else
        echo "大了一点，再试试。"
    fi
done
```

`RANDOM` 是 bash 内置的随机数变量，范围 0–32767，取模后正好落在 1–100。

## 四、三种执行方式的区别

这是作业里要求解释的重点，我总结成一张表：

| 方式 | 是否需要执行权限 | 是否新开子 shell | 脚本里的 `cd`、变量是否影响当前终端 |
| --- | --- | --- | --- |
| `./one2n` | 需要 | 是 | 不影响 |
| `bash one2n` | 不需要 | 是 | 不影响 |
| `source one2n` | 不需要 | 否 | **会影响** |

实际验证：在脚本里写 `cd /tmp` 和 `mydir=hello`，`source` 执行完之后当前终端的目录真的变了，`echo $mydir` 也能打印出 `hello`；而用 `./` 或 `bash` 执行时，当前终端毫无变化。

所以 `~/.bashrc` 这类配置文件必须用 `source`（或 `.`）执行，否则设置的 `PATH`、别名都不会生效。

## 五、把脚本放进 PATH

作业要求创建 `~/bin` 并加入 `PATH`：

```bash
mkdir -p ~/bin
mv one2n ~/bin/
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

之后在任意目录直接输入 `one2n 10` 就能运行，不必再写 `./`。

## 六、几个容易踩的坑

1. **赋值不能有空格**：`sum = 0` 会报错，必须写 `sum=0`。
2. **`[` 两边要有空格**：`[$n -gt 0]` 是错的，`[ $n -gt 0 ]` 才对。
3. **变量引用加引号**：`"$var"` 能避免变量为空或含空格时出错。
4. **`$?` 是上一条命令的退出状态**，0 表示成功，非 0 表示失败，写判断时很有用。

脚本写得多了就会发现，Shell 的语法虽然"古老"，但只要按规矩来，完成日常自动化绰绰有余。
