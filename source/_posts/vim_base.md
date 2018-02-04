---
title: vim基础
date:
modified:
categories:
- Linux
tags:
- vim
- Linux
---

1. vim编辑器在内存缓冲区中处理数据。
2. vim编辑器有两种操作模式：普通模式和插入模式。
3. vim移动光标的命令：h-左  j-下  k-上  l-右
4. 快速移动命令：PageDown、PageUp、G(最后一行)、num G(移动到第num行)、gg(移动到第一行)

### 编辑数据
x		删除当前光标所在位置的字符
dd		删除当前光标所在行
dw		删除当前光标所在位置的单词
d$		删除当前光标所在位置至行尾的内容
J		删除当前光标所在行尾的换行符
u		撤销前一编辑命令
a		在当前光标后追加数据

### 复制和粘贴
先使用dd删除，然后使用p粘贴
yw		复制一个单词
y$		复制到行尾

### 查找和替换
/content	n
:s/old/new/		替换
:s/old/new/g
:n,ms/old/new/g	替换行号n和m之间所有old
:%s/old/new/g	替换整个文件中所有old
:%s/old/new/gc	替换整个文件中的所有old，但在每次出现时提示
