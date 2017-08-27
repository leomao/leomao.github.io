+++
title = "Sync Files via SSH"
date = 2017-08-24T19:10:17+08:00
tags = ["linux", "ssh"]
+++

# Introduction

雖然之前一直說要寫點東西，但事實證明過這麼久仍然一點動靜也沒有。剛好有人問到說要怎麼 sync 本地和遠端的檔案，我就稍微查了一下，發覺這東西似乎還滿方便的，就來寫寫記錄一下好了。

<!--more-->

## Rsync + Inotifywait

想法滿簡單的，就是利用 `inotifywait` 來偵測檔案變更然後用 `rsync` 透過 ssh 做同步。要安裝 `rsnyc` 跟 `inotify-tools`。然後看了看別人寫的 [onchange.sh][onchange.sh]，我也就直接用了。不過為了方便，我稍微改寫了一下，變成直接 sync 指定的本地和遠端目錄。
[→link](https://gist.github.com/leomao/aa8b96a636a103fec063483c68ad7bac)
```console
$ watchsync.sh . host:/path/to/dir
```

## SSH Connection Mutiplexing

通常會想要持續 sync 八成都是因為會一直有連續的小改動，這種時候 overhead 反而會是在建立 ssh 連線上。查了一下發現了一個 ssh 的好用功能叫 connection mutiplexing，用途是共用同一個連線，就不需要反複重建連線了。設定的方法就是在 `~/.ssh/config` 下面加入
```
ControlMaster auto
ControlPath ~/.ssh/sockets/%r@%h-%p
ControlPersist 600
```

記得要先建好 `~/.ssh/sockets` 這個路徑。這東西會有兩個小問題，一個是如果用 sftp 或 scp 傳大檔案，會跟一般的 ssh 搶單一 connection 的 bandwidth，另一個則是 ssh tunnel 必須在建立連線的時候做設定，所以要單獨建立連線，是不能用這個選項的。因此在下 ssh 指令時要加入 `-o ControlMaster=no` 來關掉。如果懶的話可以加一些 alias 來解決這個問題。

[onchange.sh]: https://gist.github.com/evgenius/6019316

[modeline]: # ( vim: set cc=0 tw=0: )
