之前写过一篇文章[如何在 VPS 上搭建梯子](https://juejin.im/post/58d8d3cd61ff4b006cd5d83a)，那篇文章里是如果在[搬瓦工](https://bandwagonhost.com)的 VPS 搭建 `shadowsocks` 的服务进行翻墙。当初选择搬瓦工的原因是因为 VPS 很便宜，一年差不多 120RMB 的价格就可以获得一台拥有独立 IP 的 VPS 了。但搬瓦工的缺点是使用的 OpenVZ 的架构，所以无法升级 Linux 内核和使用 [Docker](https://www.docker.com) 等很多的限制。使用 Docker 虚拟机，可以自定义 `shadowsocks` 的镜像，这样现在迁移 VPS 或者给 VPS 重装系统的时候就不需要再重新搭建 `shadowsocks` 了，只需要将之前准备好的 `shadowsocks` docker 镜像下载下来并启动就可以了。本篇文章就是来教大家如何自己构建一个 `shadowsocks` 的 docker 镜像并启动和使用对 `shadowsocks` 进行加速，使大家可以通畅的浏览 `youtube` 上的 1080P 视频。

## Vultr VPS 提供商
这次我们弃用了搬瓦工，理由正如上面提到的，搬瓦工使用的是 OpenVZ 的架构，而 OpenVZ 架构是不支持 Docker 和后面提到的 `TCP-BBR` 即 TCP 拥塞控制技术的。

经过比较之后，选择了 [Vultr](https://www.vultr.com/?ref=7656065) 这个 VPS 提供商。原因是 [Vultr](https://www.vultr.com/?ref=7656065) 提供基于 KVM 虚拟的 VPS，可以支持 Docker 和 TCP-BBR 技术，并且其现在有每个月只需 `2.5 美元`的选择，对于搭建梯子和个人博客搭建等日常学习来说已经绰绰有余了。

### 充值
注册了 [Vultr](https://www.vultr.com/?ref=7656065) 帐号后，必须先充值才可以进行 VPS 的选择，现在优惠活动是充值 5 美元赠送 5 美元。像之前赠送 20 美元的活动已经取消了。


![](https://user-gold-cdn.xitu.io/2017/5/2/224fbf1fd314e3ee08543554e0b86fdc)


建议使用 PayPal 进行充值，没有 PayPal 可以去网上搜一下教程，申请一个。PayPal 在支付国际业务的时候还是很方便的。

### VPS 选择
充值完成之后，就可以去选择自己需要的 VPS 了。

![](https://user-gold-cdn.xitu.io/2017/5/2/a20fc7942b6fa8bb72d556a5f76c1592)

Vultr 在全球有 15 个数据中心，大家可以通过以下的测试链接来检测以下，自己所在区域使用哪个数据中心速度最快。


> 法兰克福（德国)       https://fra-de-ping.vultr.com/vultr.com.1000MB.bin  
巴黎（法国)             https://par-fr-ping.vultr.com/vultr.com.1000MB.bin  
阿姆斯特丹（荷兰）     https://ams-nl-ping.vultr.com/vultr.com.1000MB.bin  
伦敦（英国）          https://lon-gb-ping.vultr.com/vultr.com.1000MB.bin  
纽约（美国）          https://nj-us-ping.vultr.com/vultr.com.1000MB.bin  
芝加哥（美国）        https://il-us-ping.vultr.com/vultr.com.1000MB.bin  
亚特兰大（美国）       https://ga-us-ping.vultr.com/vultr.com.1000MB.bin  
迈阿密（美国）         https://fl-us-ping.vultr.com/vultr.com.1000MB.bin  
达拉斯（美国）         https://tx-us-ping.vultr.com/vultr.com.1000MB.bin  
西雅图（美国）          https://wa-us-ping.vultr.com/vultr.com.1000MB.bin  
硅谷（美国）        https://sjo-ca-us-ping.vultr.com/vultr.com.1000MB.bin  
洛杉矶（美国）       https://lax-ca-us-ping.vultr.com/vultr.com.1000MB.bin  
悉尼（澳大利亚）       https://syd-au-ping.vultr.com/vultr.com.1000MB.bin  
东京（日本）          https://hnd-jp-ping.vultr.com/vultr.com.1000MB.bin  
新加坡             https://sgp-ping.vultr.com/vultr.com.1000MB.bin  


直接将链接输入到浏览器里，根据下载速度来进行比较，不是东京的就一定是最快的，因为这还要考虑到你所在地区的网络情况。

选择完数据中心之后就是选择操作系统了。这个就引人而已了，我习惯使用 Debian 7 x64 的系统故依旧选择了那个。

一切选择完毕，点击 `Deploy` 按钮，你的第一个 VPS 就选择完毕，Vultr 就会根据你的选择去对应的数据中心找开辟一个 VPS 并安装上你所选择的操作系统。

### 访问 VPS
VPS 选择完毕后，就可以通过 `ssh` 需要来访问 VPS 进行一些环境的搭建了。

![](https://user-gold-cdn.xitu.io/2017/5/2/afd42f0817577397f11dd3cdb673286f)

点击 `Server Details` 查看 VPS 的详情

![](https://user-gold-cdn.xitu.io/2017/5/2/624d1328bad8e86e1931e67d23c6ed94)

如上图所示，可以得到，该 VPS 的 IP 地址和初始的用户名和密码。根据这些信息就可以访问 VPS 了。

在命令行中输入

```bash
$ ssh $Username@$IPAddress
```
将上面的 `$Username` 和 `$IPAddress` 替换为上面的红框标记的数值后，敲击回车就会提示输入密码，输入红框里的密码，如果命令行前显示
```bash
root@vultr:~#
```
说明已经成功访问到 VPS 了。下面就可以在 VPS 里搭建 Docker 服务，然后制作 `shadowsocks` 的镜像了。

### 添加用户
> 此节是针对 Linux 小白，高手请绕行。

当你登入到 VPS 的时候，默认是以 `root` 账户登入。`root` 拥有最高的权限，一直使用超级权限用户来操作是非常危险的，Linux 可没有还原的功能，所以一般情况下需要用户自己创建一个无特权的新用户来进行一些基本操作。

**添加新用户**
```bash
$ adduser $NewUser
```
将 `$NewUser` 替换为你想要的用户名，输入完命令后，会相继的让你输入新密码及一些信息，这些都输入完后就创建了一个无特权的新用户，默认这些新用户只能操作该用户目录下的文件，无权操作系统文件

之后可以通过两种方式来进行账户的切换
一种是使用 `ssh` 登入的时候可以直接使用该用户登入
```bash
$ ssh $NewUser@$IPAddress
```

另外一种是使用 `su` 命令，可以从当前用户转到对应的用户
```bash
$ su $NewUser
```

但是有的时候普通的用户也需要操作系统级文件，那就需要给当前用户授权，这样该用户就可以使用 `sudo` 命令来操作系统级的文件和执行系统级的指令。

Debian 默认并不支持 `sudo`， 需要使用 `root` 账户 安装 `suodo`
```bash
$ apt-get update && apt-get install sudo
```
之后继续使用 `root` 用户通过 `visudo` 命令来修改授权文件
```bash
$ visudo
```
找到文件中标有 `User privilege specification` 的段落，如下
```bash
# User privilege specification
root    ALL=(ALL:ALL) ALL
```
然后参照 `root` 段落添加
```bash
# User privilege specification
root    ALL=(ALL:ALL) ALL
$newuser    ALL=(ALL:ALL) ALL
```
将 `$newuser` 替换为上面新创建的用户名称，然后保存

之后在 `newuser` 的帐号下，就可以使用 `sudo` 命令来执行系统级的操作

如果要删除一个用户，也非常简单
```bash
$ sudo deluser --remove-home $username
```
如果是 `root` 用户，则不需要添加 `sudo`

## Docker 
Docker 使用 Google 公司推出的 Go 语言 进行开发实现，基于 Linux 内核的 cgroup，namespace，以及 AUFS 类的 Union FS 等技术，对进程进行封装隔离，属于操作系统层面的虚拟化技术。由于隔离的进程独立于宿主和其它的隔离的进程，因此也称其为容器。 

Docker 在容器的基础上，进行了进一步的封装，从文件系统、网络互联到进程隔离等等，极大的简化了容器的创建和维护。使得 Docker 技术比虚拟机技术更为轻便、快捷。

下面的图片比较了 Docker 和传统虚拟化方式的不同之处。传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

![](https://user-gold-cdn.xitu.io/2017/5/2/80e6a14382f46bb0e64064b18b04c1e0)

![](https://user-gold-cdn.xitu.io/2017/5/2/bec7d6bdbea103ab11cd29760ae59edc)

关于 Docker 这里就不做更详细的介绍，因为这不是本篇文章的重点，感兴趣的朋友可以查看 [Docker 从入门到实践](https://www.gitbook.com/book/yeasy/docker_practice)。

### 基本概念
Docker 包括三个基本概念
- 镜像 (Image)
- 容器 (Container)
- 仓库 (Repository)

#### Docker 镜像
Docker 镜像就是一个只读的模板。

例如：一个镜像可以包含一个完整的 ubuntu 操作系统环境，里面仅安装了 Apache 或用户需要的其它应用程序。

镜像可以用来创建 Docker 容器。Docker 提供了一个很简单的机制来创建镜像或者更新现有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用。

#### Docker 容器
Docker 利用容器来运行应用。

容器是从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。可以把容器看做是一个简易版的 Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

> 注：镜像是只读的，容器在启动的时候创建一层可写层作为最上层。

#### Docker 仓库
仓库是集中存放镜像文件的场所。有时候会把仓库和仓库注册服务器（Registry）混为一谈，并不严格区分。实际上，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。

最大的公开仓库是 [Docker Hub](https://hub.docker.com/)，存放了数量庞大的镜像供用户下载。 国内的公开仓库包括 [Docker Pool](http://www.dockerpool.com/) 等，可以提供大陆用户更稳定快速的访问。

> 注：Docker 仓库的概念跟 Git 类似，注册服务器可以理解为 GitHub 这样的托管服务。

只要理解好以上 Docker 的三个基本要素，对于 Docker 的使用就够用了。

### 安装
本篇文章只讲述如何在 Debian 系统里安装 Docker 其他系统请直接查阅[官方安装文档](https://docs.docker.com/engine/installation/)

#### 操作系统为 Debian 7 的用户请注意
操作系统为 Debian 7 的用户需要在安装 Docker 之前满足以下条件

##### 确认 Linux 内核至少为 3.10 的版本
Debian Wheezy(7) 默认是使用 3.2 的 Linux 内核，所以需要先检测一下当前内核版本号，看是否需要升级。

```bash
$ uname -r

3.2.0-4-amd64
```
如果结果低于 3.10 则需要进行内核升级，则需要通过以下的方法来对 Linux 内核进行升级

首先添加 `backports` 在 `sources.list` 文件中

```bash
$ echo "deb http://ftp.debian.org/debian/ wheezy-backports main" | sudo tee -a /etc/apt/sources.list
```
root 权限用户，请自行去掉 `sudo` ，然后更新源

```bash
$ sudo apt-get update
```
现在就可以查看一下有哪些可用的针对于该 VPS 的架构有哪些可用的 Linux 内核

```bash
$ sudo apt-cache search linux-image

alsa-base - ALSA driver configuration files
linux-headers-3.2.0-4-amd64 - Header files for Linux 3.2.0-4-amd64
linux-headers-3.2.0-4-rt-amd64 - Header files for Linux 3.2.0-4-rt-amd64
linux-image-3.2.0-4-amd64 - Linux 3.2 for 64-bit PCs
linux-image-3.2.0-4-amd64-dbg - Debugging symbols for Linux 3.2.0-4-amd64
linux-image-3.2.0-4-rt-amd64 - Linux 3.2 for 64-bit PCs, PREEMPT_RT
linux-image-3.2.0-4-rt-amd64-dbg - Debugging symbols for Linux 3.2.0-4-rt-amd64
linux-image-2.6-amd64 - Linux for 64-bit PCs (dummy package)
linux-image-amd64 - Linux for 64-bit PCs (meta-package)
linux-image-rt-amd64 - Linux for 64-bit PCs (meta-package), PREEMPT_RT
linux-headers-3.16.0-0.bpo.4-amd64 - Header files for Linux 3.16.0-0.bpo.4-amd64
linux-image-3.16.0-0.bpo.4-amd64 - Linux 3.16 for 64-bit PCs
linux-image-3.16.0-0.bpo.4-amd64-dbg - Debugging symbols for Linux 3.16.0-0.bpo.4-amd64
linux-image-amd64-dbg - Debugging symbols for Linux amd64 configuration (meta-package)
```
发现里面有 `3.16.0` 的版本正好是我们需要的，在安装升级内核之前，最好从 `backports` 源处对该系统中的一些软件进行更新，以防在安装中出现依赖包的版本错误。

```bash
$ sudo apt-get -t wheezy-backports upgrade
```
更新完成后，就可以选择 Linux 内核的版本进行升级了

```bash
$ sudo apt-get -t wheezy-backports install linux-image-3.16.0-0.bpo.4-amd64
```
之后重启 VPS 就完成了 Linux 内核的升级，重启看查看一下结果

```bash
$ uname -r
3.16.0-0.bpo.4-amd64
```
现在 Linux 内核达到了 `3.10` 之上，可以继续 Docker 的安装了

#### 设置 Docker 仓库
在第一次在本 VPS 上安装 Docker 之前，需要先设置 Docker 的仓库，之后就可以基于该仓库进行 Docker 的安装和升级。

**For Jessie (Debian 8)**
```bash
$ sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common
```

**For Wheezy (Debian 7)**
```bash
$ sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     python-software-properties
```
为了确认所下载软件包的合法性，需要添加 Docker 官方软件源的 GPG 密钥。
```bash
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```
验证的密钥 ID为: `9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88`

使用如下命令可以看到该密钥

```bash
sudo apt-key fingerprint 0EBFCD88

pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22
```
这样从 Docker 官方软件源下载软件包就是合法的了。

然后需要以下命令来设置 Doecker 的 `stable` 仓库

**For amd64:**
```bash
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```

**For armhf:**
```bash
$ echo "deb [arch=armhf] https://download.docker.com/linux/debian \
     $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list
```

 **Debian Wheezy 又需要注意了**  
Wheezy 上的 `add-apt-repository` 会自动在 apt 源中添加一个不存在的 `deb-src` 源, 需要在 `sources.list` 文件中找到并注释或删除它。否则在运行 `update` 命令时会出现错误，如下所示。

```bash
$ sudo apt-get update

W: Failed to fetch https://download.docker.com/linux/debian/dists/wheezy/Release 
Unable to find expected entry 'stable/source/Sources' in Release file (Wrong sources.list entry or malformed file)
```
所以需要修改 `/etc/apt/sources.list` 文件，将文件中的注释掉或删除掉
```xml
deb-src [arch=amd64] https://download.docker.com/linux/debian wheezy stable
```

#### 安装 Docker
首先更新以下 `apt` 包索引
```bash
$ sudo apt-get update
```

之后安装最新的 Docker
```bash
$ sudo apt-get install docker-ce
```
#### 启动 Docker
**For Wheezy**
```bash
$ sudo service docker start
```

**For Jessie**
```bash
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

#### 建立 docker 用户组
默认情况下，`docker` 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 `root` 用户。因此，更好地做法是将需要使用 `docker` 的用户加入 `docker` 用户组。

建立 docker 组
```bash
$ sudo groupadd docker
```

将当前用户加入 docker 组：
```bash
$ sudo usermod -aG docker $USER
```

#### 卸载 Docker
1. 卸载 Docker 包
```bash
$ sudo apt-get purge docker-ce
```

2. 镜像，容器等配置不会自动卸载，所以如果要删除干净，就需要手动来删除 docker 目录
```bash
$ sudo rm -rf /var/lib/docker
```
> 以上就是整个 Docker 的安装过程，主要是针对 Debian 系统，如需其他系统请查阅[这里](https://docs.docker.com/engine/installation/#supported-platforms)

## 使用 Dockerfile 定制 Shadowsocks 镜像
`ss` 服务就不多说了，下面主要是介绍 `shadowsocks-libev` 的安装过程。 Github 中有很多版本的 `ss` 开源项目，包含用 python，go 等语言写的，这里用的是 c 语言版本的 `shadowsocks-libev`。

使用该版本主要是基于两个原因:
- c 语言的执行效率高，内存消耗少
- 该项目作者一直在维护着，出现问题提 issue 修复效率高

镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

本文只会提及在定制 `shadowsocks` 镜像所使用到的几个指令，关于 Dockerfile 更多指令的详细介绍请参考[这里](https://yeasy.gitbooks.io/docker_practice/content/image/build.html)

### FROM 指定基础镜像
所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。基础镜像是必须指定的，而 FROM 就是指定基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。我将使用 Debian Jessie(8) 来作为基础镜像，所以 Dockerfile 的第一条指令就是
```dockerfile
FROM debian:jessie
```
当 `doeker` 运行 Dockerfile 的时候，这条指令会让 `docker` 会现在本地查找是否存在 `Debian jessie` 的镜像，如果本地没有存在，则去 [Docker Hub](https://hub.docker.com/explore/) 将其下载到本地，然后基于该镜像来进行定制。

### MAINTAINER 指明维护者
该指令，不会对镜像进行任何的操作，只是会指明该 Dockerfile 的文件是由谁在维护，这样日后如果有问题，使用者可以联系到维护者进行修改或优化。

```dockerfile
FROM debian:jessie
MAINTAINER Thomas <zhgqthomas@gmail.com>
```

### RUN 执行命令
`RUN` 指令是用来执行命令行命令的。由于命令行的强大能力，`RUN` 指令在定制镜像时是最常用的指令之一。其格式有两种：

1. shell 格式：`RUN <命令>`，就像直接在命令行中输入的命令一样。如

```dockerfile
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

2. exec 格式：`RUN ["可执行文件", "参数1", "参数2"]`，这更像是函数调用中的格式。

既然 RUN 就像 Shell 脚本一样可以执行命令，那么我们是否就可以像写 Shell 脚本一样把每个命令对应一个 RUN。

```dockerfile
FROM debian:jessie
MAINTAINER Thomas <zhgqthomas@gmail.com>

# add Debian jessie backports
RUN echo "deb http://ftp.debian.org/debian jessie-backports main" \
	| tee /etc/apt/sources.list.d/backports.list \
# update repository & upgrade dependencies & install shadowsocks-libev
	&& apt-get update \
	&& apt-get -t jessie-backports install -y shadowsocks-libev

```

`RUN` 运行的所有的指令只有一个目的，就是搭建和安装 `shadowsocks-libev`。其实也可以每一行指令对应一个 `RUN`。但是 Dockerfile 中每一个指令都会建立一层，`RUN` 也不例外。 这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。

### ENTRYPOINT 入口点
`ENTRYPOINT` 字面解释起来比较复杂，先直接上代码。

```dockerfile
FROM debian:jessie
MAINTAINER Thomas <zhgqthomas@gmail.com>

# add Debian jessie backports
RUN echo "deb http://ftp.debian.org/debian jessie-backports main" \
	| tee /etc/apt/sources.list.d/backports.list \
# update repository & upgrade dependencies & install shadowsocks-libev
	&& apt-get update \
	&& apt-get -t jessie-backports install -y shadowsocks-libev

# Configure container to run as an executable
ENTRYPOINT ["/usr/bin/ss-server"]

```
`ENTRYPOINT` 的目的和命令行是一样，都是在指定容器启动程序及参数。指明了 `ENTRYPOINT` 这样就可以将 `ss-server` 需要的参数通过 `docker run` 的参数进行指定。具体的用法请查看[这里](https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/entrypoint.html)

至此，Dockerfile 就编写完成了。原文件可以在这里进行[下载](https://github.com/zhgqthomas/docker-shadowsocks)

### 生成镜像
通过以上的几步，就将 `shadowsocks` 的定制镜像的 Dockerfile 编写完成。在相同目录下运行
```bash
$ sudo docker build -t shadowsocks .
```
Docker 就会根据当前目录下的 Dockerfile 来生成以 `shadowsocks` 命名的镜像了。有了镜像，不管你的 VPS 是什么系统，都可以通过以下命令来开启 `shadowsocks` 服务

```bash
$ sudo docker run --name ss -d -p 1989:1989 -p 1989:1989/udp shadowsocks -s 0.0.0.0 -p 1989 -k zhgqthomas.com -m aes-256-cfb
```
将 `$password` 替换为自定义的密码，然后 `ss` 服务就运行起来了。

可以通过以下命令来查看，`ss` 服务是否启动成功

```bash
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                            NAMES
771f572d7c49        shadowsocks         "/usr/bin/ss-serve..."   3 seconds ago       Up 3 seconds        0.0.0.0:1989->1989/tcp, 0.0.0.0:1989->1989/udp   ss
```

### 各平台 shadowsocks 客户端
- [Window](https://github.com/shadowsocks/shadowsocks-windows)
- [Mac](https://github.com/shadowsocks/ShadowsocksX-NG)
- [Android](https://github.com/shadowsocks/shadowsocks-android)
- [iOS](https://itunes.apple.com/us/app/wingy-http-s-socks5-proxy-utility/id1178584911?mt=8)

## 开启　TCP BBR　拥塞控制算法
BBR 目的是要尽量跑满带宽, 并且尽量不要有排队的情况, 效果并不比锐速差

> TCP-BBR 和锐速一样，不支持 Openvz，查看本教程之前，请先确定你的 VPS 的虚拟化技术！

需要将 Linux 内核升级到 4.9+, 如何升级内核请参考安装 Docker 小节中给 Debian 7 升级内核的方法。

### 开启
> 注意需 root 权限方可开启

执行
```bash
$ echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
$ echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```
然后保存生效
```bash
$ sudo sysctl -p
```
检查是否开启了 BBR
```bash
$ sudo sysctl net.ipv4.tcp_available_congestion_control
net.ipv4.tcp_available_congestion_control = bbr cubic reno
```
```bash
$ sudo sysctl net.ipv4.tcp_congestion_control
net.ipv4.tcp_congestion_control = bbr
```
```bash
$ lsmod | grep bbr
tcp_bbr                16384  17
```
如果如以上所示，说明 BBR 已成功开启

### 无图无真相
以下是在在联通 50M 宽带，油管观看 1080P 视频的截图

![](https://user-gold-cdn.xitu.io/2017/5/2/9c735f340938795521bda7a59da4146e)

### KCP 加速
对于不能使用 TCP-BBR 进行加速的朋友们，也不要灰心，可以使用 KCP 协议加速。关于 KCP 协议的详情请查看[这里](https://github.com/skywind3000/kcp)

#### 安装 KCPTun 
[KCPTun](https://github.com/xtaci/kcptun) 是 KCP 协议的一个简单应用，可以用于任意 TCP 网络程序的传输承载，以提高网络流畅度，降低掉线情况。由于 Kcptun 使用 Go 语言编写，内存占用低（经测试，在64M内存服务器上稳定运行），而且适用于所有平台，甚至 Arm 平台。

```bash
$ mkdir ~/kcptun
$ cd ~/kcptun
$ wget https://github.com/xtaci/kcptun/releases/download/v20170329/kcptun-linux-amd64-20170329.tar.gz
$ tar -zxvf kcptun-linux-amd64-20161222.tar.gz
```
KCPTun 不分系统版本，只分主系统和位数，比如 64x 就选择 `kcptun-linux-amd64-XXX.tar.gz` ，32x 就选择 `kcptun-linux-386-XXX.tar.gz` ，Centos/Debian/Ubuntu 都一样。

解压之后会发现只有两个文件： `client_linux_amd64` 和 `server_linux_amd64`，第一个是是客户端文件（linux的客户端），第二个是服务端文件。

> 一般 VPS 都是 Linxus 64 位系统所以使用 kcptun-linux-amd64-xxxx.tar.gz 中
的 `server_linux_amd64` 来开启 kcptun server 服务。  
> MacOS 请使用 kcptun-darwin-amd64-xxxx.tar.gz 中的 `client_linux_amd64` 来开启 kcptun client 服务。

kcptun 可以直接通过命令行加参数的方式进行开启，如
```bash
$ ./server_linux_amd64 -mtu 1400 -sndwnd 2048 -rcvwnd 2048 -mode fast2
$ ./client_linux_amd64 -mtu 1400 -sndwnd 256 -rcvwnd 2048 -mode fast2 -dscp 46
```
具体的参数含义请参考[官方文档](https://github.com/xtaci/kcptun)。

对于 Linux 小白来说可能有点吃力，故提供了脚本可开启和关闭服务。[脚本下载链接](https://github.com/zhgqthomas/kcpsh)

> 注意：客户端和服务端的参数 -sndwnd 2048 -rcvwnd 2048 这两个值不要大于你的本地宽带，否则流量消耗会浪费好几倍。适用大部分 ADSL 接入（非对称上下行）的参数。
其它带宽请按比例调整，比如 50M ADSL，把 -sndwnd -rcvwnd 减掉一半。100M 就是 2048，50M 就是 1024。这两个值可以逐渐调小，但是不能比本地的实际宽带大！
注意：产生大量重传时，一定是窗口偏大了

#### 故障排除
> 客户端和服务器端皆无 stream opened 信息。  
连接客户端程序的端口设置错误。


> 客户端有 stream opened 信息，服务器端没有。  
连接服务器的端口设置错误，或者被防火墙拦截。

> 客户端服务器皆有 stream opened 信息，但无法通信  
上层软件的设定错误。

注意 KcpTun 有个缺点，就是实际流量消耗 最少是 你使用量的两倍！如果参数调整有问题，可能会浪费十几倍的流量，而加速幅度也并不会上升多少。

## 开源
- [Dockerfile](https://github.com/zhgqthomas/docker-shadowsocks)
- [Docker Image](https://hub.docker.com/r/zhgqthomas/shadowsocks/)
- [kcpsh](https://github.com/zhgqthomas/kcpsh)
