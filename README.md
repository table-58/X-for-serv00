>鸣谢 fscarmen2 的开源仓库 [fscarmen2/Argo-Xray-JS-PaaS](https://github.com/fscarmen2/Argo-Xray-JS-PaaS) ，本仓库是基于此仓库的基础进行了 FreeBSD 相关命令的替换，并根据 Serv00 的实际环境进行了部分功能的删减与增加。
----
>**直连版本：[X-for-Serv00 Direct Branch](https://github.com/k0baya/X-for-serv00/tree/direct)**
>
>直连版本实现 Vless、VMess、Trojan 三协议并存，由于不走 Cloudflare CDN，项目特点与 Main 分支不同，部署方法也略有区别，请按需选择。
----
## X-for-Serv00

>~~项目特点与 Argo-Xray-JS-PaaS 基本一致，但是由于 Serv00 端口数量限制，本仓库仅实现了 vmess 协议，并删除了 webssh 以及 webftp 以及探针的功能。~~
>
>~~而且本仓库流量通过 Warp 增加了 IPv6 支持。并在源仓库的基础上添加了订阅功能，在`/list`路径列表下可以看到订阅地址，如果不固定 ARGO 隧道，也可以通过订阅地址获取最新配置，而不需要打开网页查看。~~

新版本做出了较大修改，目前实现 Vless、Vmess、Trojan 三协议并存，去除了临时隧道的使用，仅支持固定隧道使用。故不再支持订阅功能。

流量是由客户端经由 Cloudflare CDN 转发到 Serv00，再由 Xray 判断是否通过 Warp 转发到目标。支持 IPv6。
### 部署
#### 准备工作

首先你需要至少拥有一个托管在 Cloudflare 的域名。

然后参考[群晖套件：Cloudflare Tunnel 内网穿透中文教程 支持DSM6、7](https://imnks.com/5984.html)的教程，在 Cloudflare 控制面板中创建一个 Argo Tunnel，把其中 ey 开头的一串 Token 记录下来备用。

同时你还需要一个 Serv00 的账号。

#### 部署 X-for-Serv00

SSH 登录 Serv00，输入以下命令：
```shell
bash <(curl -s https://raw.githubusercontent.com/k0baya/X-for-serv00/main/entrypoint.sh)
```
并按照提示输入相关信息。

>**注意**
>
>其中 `ARGO_AUTH`、`ARGO_DOMAIN_VL`、`ARGO_DOMAIN_VM`、`ARGO_DOMAIN_TR` 四个变量是必须的，其他变量根据需要填写。非必须的变量可以按回车跳过，使用默认值。
>
>`ARGO_AUTH` 为上述的 ey 开头的一串 Token。
>
>`ARGO_DOMAIN_VL`、`ARGO_DOMAIN_VM`、`ARGO_DOMAIN_TR` 分别为 Vless、VMess、Trojan 协议的域名，请使用你域名的子域进行设置，如下图：![](/pic/argo.png)


#### 启动并获取配置

按照脚本提示进入 `/status` 的网页，并尝试刷新页面，直到进程列表中出现了包含 `web.js` 以及 `cloudfalred` 字样的进程，就代表 X-for-Serv00 已经启动成功。此时你就可以通过访问 `/list` 路径查看到 X-for-Serv00 所提供的配置链接了。

### 自动启动

此次版本更新之后，X-for-Serv00 已经可以摆脱 Serv00 的 Crontab 启动，你可以通过访问网页对项目进行唤醒，如果你需要保活，可以使用以下公共服务对网页进行监控：

1 [cron-job.org](https://console.cron-job.org)

2 [UptimeRobot](https://uptimerobot.com/) 

同时，你也可以选择自建 [Uptime-Kuma](https://github.com/louislam/uptime-kuma) 等服务进行监控。

### 更换 Cloudflare CDN 接入点

目前本仓库使用的默认 Cloudflare CDN 接入点为 `upos-sz-mirrorcf1ov.bilivideo.com:443` ，如果这个接入点在你的网络中表现不佳，你可以选择使用 [XIU2/CloudflareSpeedTest](https://github.com/XIU2/CloudflareSpeedTest) 或者其他优选 IP / 域名 替换导出的配置中的 Cloudflare CDN 接入点。**注意，只需要修改客户端的配置，不要修改已经部署的 X-for-Serv00 项目。**