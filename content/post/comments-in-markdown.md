+++
title = "Comments in Markdown"
date = 2017-08-24T21:40:48+08:00
categories = []
tags = []
+++

Markdown 似乎沒有註解的語法，隨意查了一下後發現一個簡單的方法：

```markdown
[//]: # ( This is a comment )
```

簡單來說就是建一個沒用的 link reference 來當作註解啦。這個方法可以用來設定 Vim 的 modeline，也算是解決了一個小問題。範例：

```markdown
[modeline]: # ( vim: set cc=0 tw=0: )
```

[modeline]: # ( vim: set cc=0 tw=0: )
<!--more-->
