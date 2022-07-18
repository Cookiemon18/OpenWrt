# 自用 OpenWrt 分享

感谢 `OpenWrt` 项目组，以及众多开发者

> 声明：此固件为个人自用，不会听从任何添加插件的建议（除非我需要），需要其他第三方插件可自行拉取官方源码并切换至 `21.02.3` 分支后自行前往 `Github` 寻找插件并且添加
>
> **此固件不适合新手**
>
> 默认网关：`10.0.0.1`
>
> 自动绑定网口默认将：最后一个网口作为 `WAN` 口，其余接口即 `eth0 eth1 eth....` 作为 `LAN` 口 [更多](#默认网口绑定)
>
> 此固件的插件基本做到了开箱即用，无需其他配置。且对部分插件做了归类处理（全挤在服务里看着太恶心了）
>
> 固件默认扩容 5G 和 10G 自行选择

![image-20220717212921695.png](https://github.com/Cookiemon18/OpenWrt/blob/IMAGE/image-20220717212921695.png)

此固件包含功能

+ `Https访问` ：因为个人有外网服务需求，所以 `luci` 界面使用了 `nginx-ssl` [更多](#Nginx 管理)

+ `Openssh`：此固件并没有使用选择使用 `dropbear` 而是选择 `OpenSSH` [更多](#OpenSSH)

+ `WireGuart`

+ `KMS激活服务器`

+ `微信推送`

+ `iKoolProxy`：去广告 [虽然我对去广告没有需求]

+ `AdGuard Home`

+ `阿里DDNS`：此固件仅支持 `阿里ddns`

+ `网络唤醒`

+ `UPnP`

+ `SAMB`：用户名 `smb`、`admin`、`openwrt` 密码 `openwrt.openwrt` [更多](#Samba4)

+ `Alist`

+ `阿里网盘 WebDAV`

+ `Aria2`：[更多](#Aria2)

+ `Open可拉斯`

+ `帕斯Wall`

+ **此固件并没有开启 `IPV6` 的支持（因为我用不到）**

![image-20220717223849139](https://github.com/Cookiemon18/OpenWrt/blob/IMAGE/image-20220717223849139.png)

# 使用说明

----

## OpenSSH

因为使用了 `OpenSSH` 所以需要先设置系统密码后才能使用密钥登陆 （可在控制台设置，也可到后台使用 `passwd` 设置）

因为开启了公网 22 端口访问 （需要手动打开防火墙入站）所以默认关闭密码登陆，使用密钥登陆。

因为不会 `JS` 无法做图形界面适配，其余配置自行前往 `/etc/ssh/sshd_config` 配置

为了方便第一次使用。我使用了 `SSHkey` 的源码进行修改实现了在 `Web` 管理界面添加密钥，效果如下

![image-20220717213643641](https://github.com/Cookiemon18/OpenWrt/blob/IMAGE/image-20220717213643641.png)

## Nginx 管理

默认关闭公网服务 `Web` 管理界面，但是我配置好了一个名为`_wan` 的服务且配置了自签证书 (需要 `AC` 证书的话自行研究)，默认关闭此服务

有需要自行前往 `/etc/config/nginx` 中将 `config server '_wan'`中的 `list listen '0.0.0.0:443'` 中的 `443` 修改成你需要对外暴露的端口然后使用 `ssh` 连接至后台使用 `uci set nginx._wan=server && uci coomit nginx` 命令开启 (但是我并不建议这么做，应该尽量减少对公网暴露的端口，建议使用 `WireGuard` 建立连接)

关于 `nginx` 的更多配置请参考[官方文档](https://openwrt.org/docs/guide-user/services/webserver/nginx)

## Samba4

我给 `samba` 设置了默认的用户名和密码分别为 

用户名：`smb` 

密码： `openwrt.openwrt`

且做了用户名映射，默认只映射了 `admin`、`openwrt` 即可以使用这两个用户名登陆 `smb` 用户，如果需要用户名，自行前往 `/etc/samba/smbusers` 添加用户名。用户名之间使用空格区分

默认挂载了 `/root/Downloads` 目录，且需要用户名与密码登陆

对于共享文件我使用了一个组，组名叫 `public_file` 只有将用户添加进这个组，且将要共享的文件夹所有组设置为这个组才能进行服务，一切权限问题都是访问权限在作妖，请自行研究

## Aria2

我这边默认配置好了下载目录以及自签证书，开箱即用，如果需要修改下载目录请确认 `aria2` 用户对你的目录具有权限（当然你还可以直接使用 `root` 用户运行但是不建议），具体还是与前面的 `Samba` 一样，自行研究

因为开启了 `Https` 所以使用 `ariang` 前需要先将 `https://10.0.0.1:6800/jsonrpc` 添加到浏览器的证书验证例外，可以试试直接打开链接然后如图操作

![image-20220717223119281](https://github.com/Cookiemon18/OpenWrt/blob/IMAGE/image-20220717223119281.png)

## 默认网口绑定

为了方便使用，我默认将最后一个网口绑定为`WAN`口，其余网口全部为 `LAN` 口

如果需要使用后台修改配置文件的话，`OpenWrt-21.02.3` 的 `/etc/config/network` 配置文件与之前有所差别（虽然官方文档说支持旧版的配置文件，但还是跟上主流）

新的配置文件为

``````bash
config interface 'loopback'
        option device 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'

config device
        option name 'br-lan'
        option type 'bridge'
        list ports 'eth0' 
        list ports 'eth1' # 这样添加网口而不是在 ‘lan’ 添加
        list ports 'eth2'
        list ports 'eth3'
        list ports 'eth4'
        list ports 'eth5'
        option ipv6 '0'

config interface 'lan'
        option device 'br-lan'
        option proto 'static'
        option netmask '255.0.0.0'
        option ipaddr '10.0.0.1'

``````

# 结尾

目前固件已在我的设备上稳定运行了接近半个月，因为是自用固件，如果后期没有严重问题估计应该没个一年半载不会更新。

此固件仅保证在 `J3160 + 8186 + BCM5719 + 英特尔千兆网卡` 正常运行

发行版仅在我自用固件基础上添加了其他网卡驱动 （网卡驱动这一块我把官方支持的都集成了，第三方驱动只添加了 `R8186` 如此固件无适合你的驱动，建议尝试自己编译，真的不难），没有添加任何无线支持
