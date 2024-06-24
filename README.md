# ztncui-aio

[![docker workflow](https://github.com/fredliang44/derper-docker/actions/workflows/docker-image.yml/badge.svg)](https://hub.docker.com/r/runyf/ztncui-aio)
[![docker pulls](https://img.shields.io/docker/pulls/runyf/derper.svg?color=brightgreen)](https://hub.docker.com/r/runyf/ztncui-aio)
[![platfrom](https://img.shields.io/badge/platform-amd64%20%7C%20arm64-brightgreen)](https://hub.docker.com/r/runyf/ztncui-aio/tags)


ZeroTier当前版本: 1.14.0

## 关于ztncui作者

对他们的工作表示衷心的感谢!

### ZeroTier 网络控制器用户界面-Docker容器

这是一个构建 **[ZeroTier One](https://www.zerotier.com/download.shtml)** 和 **[ztncui](https://key-networks.com/ztncui)** 的Docker镜像，用于启动 **standalone ZeroTier network controller** 带有用户管理界面的容器

关于key_networks [![alt @key_networks on Twitter](https://i.imgur.com/wWzX9uB.png)](https://twitter.com/key_networks)

Licensed Under GNU GPLv3

## 构建你自己的镜像

支持 aarch64 (arm64/v8), amd64 (默认)

Armv7(means armhf) 应该也可以, 但是没有经过测试

其它架构不支持

```bash
$ git clone https://github.com/runyf/ztncui-aio
$ docker build . --build-arg OVERLAY_S6_ARCH=<one of aarch64,x86_64> -t ghcr.io/kmahyyg/ztncui-aio:latest
```

> 为什么不知道识别系统架构? 有些内核提供的查询方法可能不标准

可以更改 `NODEJS_MAJOR`变量在Dockerfile以便使用不退的 nodejs 版本

一定不要使用`node_lts.x` 作为你的nodejs的安装脚本，它的版本可能会因为时间的变化而改变而没有进一步的通知

## 使用

### Golang auto-mkworld (已经内置在容器中)

这个特性允许你在不使用编译器的情况下生成行星文件   

另外，由于zero - tier- one UI的IPC限制和多种问题，我们不支持自定义端口，这里只能使用9993/udp端口。   

在创建容器时，根据需要设置以下环境变量:

| 必需 | 参数名| 描述 | 默认值 |
|:--------:|:--------:|:--------:|:--------:|
| no | AUTOGEN_PLANET | 如果将值设置为1将自动生成planet和moon文件。默认为0，它将使用官方的planet的文件，就是说使用这个值搭建出来的仅仅是一个网络控制器+UI。 如果设置为2, 则使用此配置 `/etc/zt-mkworld/mkworld.config.json`.  | 0 |

参考配置文件可以在“ztnodeid/assets/mkworld.conf.json”中找到。

您也可以自己定义，并检查stdout输出以获得自定义行星文件。之后，您将在http文件服务器根目录下找到自定义行星文件和ca证书。

JSON配置描述:

```json
{
    "rootNodes": [   // 节点数组，可以是多个
        {
            "comments": "amsterdam official",   // 节点对象，如果AUTOGEN_PLANET=1，将自动生成
            "identity": "992fcf1db7:0:206ed59350b31916f749a1f85dffb3a8787dcbf83b8c6e9448d4e3ea0e3369301be716c3609344a9d1533850fb4460c50af43322bcfc8e13d3301a1f1003ceb6",  
            // node identity.public ^^ , 如果节点没有初始化，将在容器启动时初始化
            "endpoints": [
                "195.181.173.159/443",   // 如果AUTOGEN_PLANET=1，将自动生成格式为ip/port的值
                "2a02:6ea0:c024::/443"   // 必须小于或等于两个端点，一个用于IPv4，一个用于IPv6。如果有多个IP，请设置多个不同身份的节点。
            ]
        }
    ],
    "signing": [
        "previous.c25519",   // planet签名密钥，如果不存在，将生成
        "current.c25519"   // 同上，用于迭代和更新
    ],
    "output": "planet.custom",   // 输出文件名
    "plID": 0,    // 行星数字ID，如果你不知道，不要修改，并设置plRecommend为true
    "plBirth": 0,  // 行星创建时间戳，如果不知道，请勿修改，并将plRecommend设置为true
    "plRecommend": true  // 设置plRecommend为true时，自动生成plID、plBirth的值。要了解更多细节，请阅读zerotier-one官方repo中的mkworld源代码
}
```

### Docker image

```bash
$ docker pull runyf/ztncui-aio
$ docker run -d -p3180:3180  -p3000:3000 -p9993:9993/udp \
  -v ~/ztncui/etc:/opt/key-networks/ztncui/etc \
  -v ~/ztncui/zt1:/var/lib/zerotier-one \
  -v ~/ztncui/zt-mkworld-conf:/etc/zt-mkworld \
  -e ZTNCUI_PASSWD=password \
  -e AUTOGEN_PLANET=1 \
  -e PLANET_RETR_PUBLIC=true \
  -e HTTP_ALL_INTERFACES=true \
  --restart always \
  --cap-add=NET_ADMIN --device /dev/net/tun:/dev/net/tun \
  --name ztncui \
runyf/ztncui-aio:v1.14.0

## 支持使用本地持久存储的配置

关于: https://github.com/key-networks/ztncui


在创建容器时，根据需要设置以下环境变量:

| 必需 | 参数名 | 描述 | 默认值 |
|:--------:|:--------:|:--------:|:--------:|
| YES | NODE_ENV | https://pugjs.org/api/express.html | production |
| no | HTTPS_HOST | HTTPS_HOST | NO DEFAULT, MEANS DISABLED |
| no | HTTPS_PORT | HTTPS_PORT | NO DEFAULT, MEANS DISABLED |
| no | HTTP_PORT | HTTP_PORT | 3000 |
| no | HTTP_ALL_INTERFACES | 使用方向代理监听所以端口, 仅HTTP | NO DEFAULT |

Note: If you do NOT set `HTTP_ALL_INTERFACES`, the 3000 port will only get listened inside container, means `127.0.0.1:3000` by default.

This application does NOT have a built-in protection mechanism against brute-force attack, you should NOT directly expose it on the internet.

And you should ALWAYS NOT use a weak password.

Set the following environment variable when create the container, and according to your needs:

| MANDATORY | Name | Explanation | Default Value |
|:--------:|:--------:|:--------:|:--------:|
| no | MYDOMAIN | generate TLS certs on the fly (if not exists) | ztncui.docker.test |
| no | ZTNCUI_PASSWD | generate admin password on the fly (if not exists) | password |
| YES | MYADDR | your ip address, public ip address preferred, will auto-detect if not set | NO DEFAULT |


**WARNING: IF YOU DO NOT SET PASSWORD, YOU HAVE TO USE `docker container logs <CONTAINER_NAME / CONTAINER_ID>` to get your random password. This is a gatekeeper.**

To reset password of ztncui: delete file under `/mydata/ztncui/passwd` and set the environment variable to the password you want, then re-create the container. After application has been initialized, the password should ONLY be changed from the web page.

## Public File Server

| MANDATORY | Name | Explanation | Default Value |
|:--------:|:--------:|:--------:|:--------:|
| no | PLANET_RETR_PUBLIC | File server listened globally or only local | NO DEFAULT |

If `PLANET_RETR_PUBLIC` is set, then file server will listen on `0.0.0.0`, otherwise, `127.0.0.1` .
This image exposed an http server at port 3180, you could save file in `/mydata/ztncui/httpfs/` to serve it. 
(You could use this to build your own root server and distribute planet file, even though, that won't hurt you, I still suggest to set a protection for both http servers in front.)

## Chinese users only

This script use https:///ip.sb for public IP detection purpose, which is blocked in some area of China Mainland. Under this circumstance, the program will try to detect public IP using `ifconfig` tool and might lead to unwanted result, to prevent this, make sure you set `MYADDR` environment variable when docker container is up.

**This repo (https://github.com/kmahyyg/ztncui-aio) only accept Issues and PRs in English. Other languages will be closed directly without any further notice. If you come from some non-English countries, use Google Translate, and state that at the beginning of the text body.**

