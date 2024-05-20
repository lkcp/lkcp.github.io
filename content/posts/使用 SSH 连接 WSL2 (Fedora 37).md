+++
title = "使用 ssh 连接 WSL2 (Fedora 37)"
author = ["lkcp"]
date = 2024-05-20T20:41:00+08:00
tags = ["WSL"]
draft = false
+++

## 使用 genie 启用 systemctl, 进而启动 ssh server {#使用-genie-启用-systemctl-进而启动-ssh-server}

Fedora 不支持 service start sshd, 使用的是基于 systemd 的系统，但是 WSL 版本中不支持 systemd. 所以可以通过 genie 使能 WSL 中的 systemd.

首先到 genie 仓库 (<https://github.com/arkane-systems/genie>) 中下载 rpm 包，随后使用 dnf install 安装这个包。 运行 genie -s 启用 systemd, 随后执行 systemctl start sshd 即可。


## 安装 tailscale {#安装-tailscale}

然后就是 WSL2 是 NAT 模式，所以没有独立的 IP, 我看到现有的方案多是基于 windows 端口转发来实现，但是这种方式有个问题就是 WSL 的 IP 再重启时会发生变化。所以我决定直接跑一个 tailscale 来给它一个我可以公网访问的 IP。刚好 tailscale 也需要 systemd, 上一步一举两得了。

安装方式见 <https://tailscale.com/download/linux/fedora>.

最终，通过 ssh username@ipaddress -p port 连接即可。
