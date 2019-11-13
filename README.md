# 通过 Windows 搭建 Gitlab

## 痛点

在搭建 Gitlab 之前，请先思考以下几个问题：为什么会选择 git 而不用 svn？应该选择哪种适合自己或团队的 git 服务器？想要搭建 Gitlab 是否因各种难题甚至想要放弃？

[不同git服务器对比](https://www.slant.co/topics/1440/~best-self-hosted-web-based-git-repository-managers)

有时候我们不想把代码托管在外网上，想在内网搭建一个 git 服务器，该如何选择呢？

可能大家会从团队人数、硬件等方面考虑，我选择 Gitlab 的原因是它有非常美观的 web 界面，而且功能很强大。但是搭建起来有点麻烦，尤其是在 Windows 上。



## 在 Windows 中搭建 Gitlab

在 Windows 中搭建 Gitlab 可能并非一件易事。

### (一) 使用Docker

第一种方法是通过`docker`，首先安装[docker for windows](https://hub.docker.com/editions/community/docker-ce-desktop-windows)，接下来在docker中搭建gitlab，[参考这里](https://docs.gitlab.com/omnibus/docker/)。

在网上有很多教程，比如这篇：[win10企业版在docker上部署gitlab](https://blog.csdn.net/MonoBehaviour/article/details/84852984)，按照教程一步一步确实是可以运行 Gitlab 并且可访问的。

然而，这种方式是存在风险的。Gitlab官方[文档](https://docs.gitlab.com/omnibus/docker/)中并不推荐，主要因为存储权限和其它一些未知问题，最严重的是它不支持持久化存储，当你的服务器重启之后，Gitlab 中的所有数据都会丢失。而且我没有找到好的解决办法。

所以，这种方法目前是**不可取**的。

### (二) 使用虚拟机

第二种方法是通过在虚拟机中运行一个 Linux 系统来搭建 Gitlab，我使用的是 [VMware](https://my.vmware.com/cn/web/vmware/info/slug/desktop_end_user_computing/vmware_workstation_pro/15_0) + [CentOS](https://www.centos.org/download/)。

该方法主要分为3步：

1. 在 VMware 中安装 CentOS 系统
2. 在 CentOS 中安装 Gitlab
3. 网络配置，使得局域网内可访问 Gitlab

下面将对这3个步骤作详细介绍——



## 步骤一：在VMware上安装CentOS

1. 打开VMware，新建虚拟机 - 典型 - 安装程序光盘映像文件
   ![1](images/1.jpg)

2. 选择CentOS系统文件
   ![2](images/2.jpg)

3. 自定义硬件

   增加内存到2G

   调整处理器数目

   网络连接可选择桥接模式，也可以选择NAT模式。在后面需进行配置

   删除声卡选项

   其它不变

   ![3](images/3.jpg)

4. 准备安装
   ![4](images/4.jpg)

5. 软件选择中，选择带GUI的服务器，这样在安装完成后会带一个界面，方便后面操作。
   ![5](images/5.jpg)

6. 设置ROOT密码，创建用户
   ![6](images/6.jpg)

7. 完成安装，重启

   ![7](images/7.jpg)

   至此，CentOS便安装成功了。

   

## 步骤二：在 CentOS 中安装 Gitlab

在 CentOS 等 Linux 系统中安装 Gitlab，可参考[官方教程](https://about.gitlab.com/install/#centos-7)。下面是一些详细说明：

### (一) SSH

安装 ssh

```shell
sudo yum install -y curl policycoreutils-python openssh-server
```

需要注意的是`policycoreutils-python`在 CentOS 8 中变成了`python3-policycoreutils`，所以目前在 CentOS 8 中是无法直接安装 Gitlab 的。

将ssh服务设为开机自启动

```shell
sudo systemctl enable sshd
```

开启ssh服务

```shell
sudo systemctl start sshd
```

### (二) 防火墙

安装防火墙，如果已安装了防火墙，则可跳过这一步

```shell
yum install firewalld systemd -y
```

开启防火墙

```shell
service firewalld start
```

将 http 和 https 添加到防火墙，permanent 表示永久生效

```shell
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
```

重启防火墙

```shell
sudo systemctl reload firewalld
```

### (三) Postfix

接下来，安装 Postfix 用来发送通知邮件。如果你想使用其它方法来发送邮件，可以跳过这个步骤，并在 Gitlab 安装完成后配置 SMTP 服务。

以下分别是安装、设为自启动以及开启。

```
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

### (四) 安装gitlab

在这里，我们需要通过 wget 来下载 Gitlab安装包，如果 wget 没有安装，则可通过下面的命令安装：

```shell
yum -y install wget
```

此外，你可能还需要安装 vim，因为后面需要修改 Gitlab的配置文件：

```
yum install vim -y
```

接下来是下载 Gitlab 安装包，到[官网](https://packages.gitlab.com/gitlab/gitlab-ce)可以下载最新的安装包。

为了保证下载速度，可以从[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/)下载，但不是最新的。

```shell
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-11.11.8-ce.0.el7.x86_64.rpm
```

安装 Gitlab

```shell
rpm -i gitlab-ce-11.11.8-ce.0.el7.x86_64.rpm
```

### (五) 修改配置

等待 Gitlab安装完成后，终端界面会提示你修改 `/etc/gitlab/gitlab.rb` 文件，通过设置 `external_url` l来配置你的 URL，通常情况下是你的服务器 ip 和端口号。

```shell
vim /etc/gitlab/gitlab.rb
```

终端界面还提示你通过下面的命令启动gitlab：

```shell
sudo gitlab-ctl reconfigure
```

到此为止，你在 CentOS 中的浏览器输入 http://localhost 或者 ip及端口号，便可以看到 Gitlab 的页面了。



## 步骤三：配置网络，使得局域网内可访问

如果你的 VMware 虚拟机网络适配器选择的是“桥接模式”，那么只需在 CentOS 中设置一个静态 IP，以后局域网内便可通过该 IP 访问 Gitlab。(也许可以不用设置静态ip，我没有试过虚拟机每次重启，ip 会不会变化)。

如果你的 VMware 虚拟机网络适配器选择的是“NAT模式”，那么需要配置网络端口转发，即将主机的端口与虚拟机的端口关联起来，今后访问主机 IP 便可间接访问到虚拟机中的 Gitlab。

关于桥接模式和 NAT 模式的区别，可参考[这篇文章](https://blog.csdn.net/ning521513/article/details/78441392)。

### (一) 设置静态IP

在 CentOS 7 中，通过以下命令编辑IP：

```shell
vim /etc/sysconfig/network-scripts/ifcfg-ens33
```

修改如下，注意其中加注释的几处

```
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static		# 使用静态IP地址，默认是dhcp
IPADDR=192.168.5.122	# 设置的静态IP
NETMASK=255.255.255.0	# 子网掩码
GATEWAY=192.168.5.2	    # 网关
DNS1=192.168.5.2		# DNS服务器
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=8df5a6e0-cba0-4f89-b55c-319d2616a985
DEVICE=ens33
ONBOOT=yes
ZONE=public
```

注意：在 NAT 模式下，这里设置的静态 IP 网关不要与主机的网关相同。

最后，重启网络使刚才配置的IP生效

```shell
service network restart
```

### (二) 配置NAT模式网络端口转发

打开 VMware 菜单，编辑 - 虚拟网络编辑器 - NAT 设置。



## 常见问题

最后，总结一下安装过程中的常见问题和解决办法

1. 虚拟机无法开启，提示 VMware Workstation 15 与 Device/Credential Guard 不兼容。
   解决办法：关闭 Hyper-V，[参考这里](https://blog.csdn.net/p942005405/article/details/89674440)。

2. yum 被其它进程占用，无法安装。
   解决办法：将该进程杀掉。

3. 局域网内其它电脑无法访问。
   解决办法：可能需要开放 Windows 端口，[参考这里](https://www.cnblogs.com/zhurong/p/9398602.html)。

4. Gitlab 中的时区不对
解决办法：[参考这里](https://www.cnblogs.com/linkenpark/p/8423358.html)。
   
5. 如何汉化？
   解决办法：[参考这里](https://gitlab.com/xhang/gitlab/wikis/home)。

   


## 参考文章

在写这篇教程时，参考了如下的一些文章：

1. [VMware 安装 CentOS + gitlab](https://www.cnblogs.com/zhouyun-yx/p/10444451.html)
2. [CentOS 7 设置静态 ip](https://blog.csdn.net/sjhuangx/article/details/79618865)









