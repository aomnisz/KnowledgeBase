# 在 Linux 后台运行脚本的三种办法

[CreateTime]: # (2020.09.09)
[ModifyTime]: # (2020.09.09)

在管理 Linux 服务器的时候，经常是有很多后台任务，退出了 ssh 之后仍要继续运行的。对于 Nginx 之类的程序还好说，它们自己会注册成系统服务器，甚至能开机运行。而如果要持续运行一些自己的小脚本的话，那就要头秃了。因为有好几种方法可以运行，这里就总结一下我知道的吧，按照我了解它们的时间顺序进行排序。

## Nohup - 最多人使用

在网上搜`在 Linux 后台运行脚本`肯定有一大堆文章是跟你说用 nohup 的。没错，我刚开始也是用这个的。原理什么的网上一大堆，我就不赘述了，这里给个[链接](https://zh.wikipedia.org/wiki/Nohup)。nohup 的命令也不难敲，就是有点难记住，尤其是后面的重定向：

```bash
nohup "command" > /dev/null 2>&1 &
```

如果想要开机就运行的话，可以把这条命令写进 `/etc/rc.local` 里；如果想要定时运行的话，就用 `crontab` 。基本上配合起来，什么事情你想到的，都能做到。但我还是会有烦恼，记不住后面那个重定向的命令，怪我自己脑子不好使吧。好在我无意中发现了一个好东西：**tmux** 。

## tmux - 像我这种小白可用

其实 tmux 的本质上是一个终端复用器，通常是用来“分屏”的，但是被我搞来当别的使用~~（其实还挺好用的）~~。nohup 的原理是让程序不接收终端退出时发出的 SIGHUP 信号，而 tmux 更厉害，直接就不会退出终端，妙啊！使用 tmux 来运行后台程序只需要 3 步：

1. 运行命令 `tmux` 打开 tmux 终端
2. 在 tmux 终端里运行你想运行的程序
3. 按组合键 `Ctrl+B D` **分离** tmux 终端

如果之后想要进入终端对程序进行什么调整的话，运行 `tmux a` 命令就好了。

多方便，我再也不用记 `2>&1` 了！

tmux 虽好，但还是有点局限性，就是不能开机启动什么的。虽然还是能曲线救国，搞个脚本什么的然后塞进去 `/etc/rc.local` 解决，但还是**麻烦**啊。

```bash
# /etc/rc.local
tmux new-session -d -s services
tmux send-keys -t services:0 'a soooo cool command' Enter
tmux new-window -t services:1
tmux send-keys -t services:1 'another cool command' Enter
```

PS. 其实这个有时候超好用的，就例如在开发的时候，用 tmux 来启动开发环境，然后 detach 到后台，有需要再 attach 到前台，舒服~

## systemd - 开机自启的核武器

~~其实我写这篇文章就为了学/记录这个东西。~~

systemd 是系统的服务，那些 Web 服务什么的通常是用这个启动的，也是用这个来维持运行的。记得它好像是把程序都挂在了 PID 为 1 的 init 进程上，还能声明说等哪个服务启动之后再启动自身。反正用它来搞开机自启就是好处多多，唯一的不好就是要写配置文件。我脑子不好老是记不得，于是就打算写在文章了，需要就来这查。

systemd 的配置文件都放在 `/usr/lib/systemd/system/` 和 `/etc/systemd/system/` 目录下（后者优先级更高）。而我通常是创建服务，也就是 `.service` 后缀。

要想注册个服务，首先要写配置文件。举个例子，我想要一个名为 drink_milk 的服务：

```
# /etc/systemd/system/drink_milk.service
[Unit]
Description=I love drink milk

[Service]
Type=simple
ExecStart=/usr/bin/drink_milk
Restart=always

[Install]
WantedBy=multi-user.target
```

这个是一份比较简单的配置文件，其中的 `Description` 和 `ExecStart` 的内容是需要修改成自定义的内容，其他可以不变。

写好之后就可以开始用起来啦。给出几个常用的命令：

```bash
systemctl start drink_milk      # 运行服务
systemctl stop drink_milk       # 停止服务
systemctl status drink_milk     # 查看服务状态
systemctl enable drink_milk     # 设置开机运行服务
systemctl disable drink_milk    # 取消开机运行服务
```

关于进阶，给几个链接吧：

- [systemd (简体中文) - ArchWiki](https://wiki.archlinux.org/index.php/systemd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
- [Systemd 入门教程：命令篇](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)
- [Systemd 入门教程：实战篇](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html)
