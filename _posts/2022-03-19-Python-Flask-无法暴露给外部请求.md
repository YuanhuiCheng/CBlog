__问题__:

我在 GCP 的 VM 上运行了 Python Flask，用的全是 default 值 （`port`：5000，`host`: localhost/127.0.0.1）。除了 Flask 后端，我还在本地电脑上运行了 React 前端，试图向后端发送 HTTP POST 请求，然而一直显示 `ERR_CONNECTION_REFUSED`，也就是请求并没有送达后端。

__解决方法__:

我运行 Flask 时用的 command 是 `flask run`，无论是在 command 里还是代码里，都没有特别声明 `host`。

如果没有声明 `host` 的话，Flask server 默认 `host` 是 `127.0.0.1`（IP 地址），也就是 `localhost`（域名）。要想 OS 能够监听到所有的公共 IP，Flask server 需要向公众暴露自己。方法就是运行时使用 command `flask run --host=0.0.0.0`。

__127.0.0.1 和 0.0.0.0 的区别__

`127.0.0.1` 这个 IP 地址被赋予给了 `loopback` 或者本地接口。只有本地的 client 才能与这个 IP 互动，监听 `127.0.0.1` 的进程也只能收到本地的连接请求。

`0.0.0.0` 指本机上所有的 IPV4 地址。如果一个服务器/OS 监听了 `0.0.0.0`，就意味着它监听了所有可用的网络接口。相应的，监听它的服务器就可以接收来自外部机器的请求。
