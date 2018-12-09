# shadowsocks服务器搭建

> 以本文以vultr centos6服务器为例

## vultr修改Root密码

1. 访问控制台，打开在线 Console，点击右上角的 “Send CtrlAltDel”。
2. 您将看到一个 GRUB 引导提示符，在 GRUB 提示符处，键入 `a` 以附加到引导命令，添加文本“single”，然后按回车；
3. 系统将启动，看到根提示后，键入 `passwd` 更改 root 密码，然后重新启动。

## 设置ssh登录

1. 在本地电脑当前用户执行`ssh-copy-id usrname@server-address`

   > usrname为服务器端的账户，如root
   >
   > server-address为服务器ip地址

2. 输入root密码确认

## 登录服务器

1. 本地执行`ssh usrname@server-address`

## 禁用密码登录

> 服务器默认开启密码登录，但网上会有人恶意穷举破解密码，禁用密码登录可以防止这种情况。
>
> 如果本地ssh密钥丢失，只能在vultr官网控制台重新开启密码登录

1. 编辑远程服务器上的`sshd_config`文件

   > `nano /etc/ssh/sshd_config`

2. 找到如下选项并修改

   ```shell
   #PasswordAuthentication yes 改为
   PasswordAuthentication no
   ```

3. 编辑保存完成后，重启ssh服务使得新配置生效，然后就无法使用口令来登录ssh了

   `service sshd restart`

## 解决SSH自动断线，无响应的问题

1. 执行以下命令

   > `echo 'ClientAliveInterval 30'  >>  /etc/ssh/sshd_config`
   >
   > `echo 'ClientAliveCountMax 30'  >>  /etc/ssh/sshd_config`

2. 其中ClientAliveInterval表示每隔多少秒，服务器端向客户端发送心跳，

   ClientAliveInterval表示上述多少次心跳无响应之后，会认为Client已经断开

## 升级centos6内核

> 由于bbr算法要求内核版本大于4.9，这里先升级内核

1. 安装最新内核

```shell
#导入ELRepo 公钥
wget https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm --import RPM-GPG-KEY-elrepo.org
#安装ELRepo
rpm -Uvh http://www.elrepo.org/elrepo-release-6-8.el6.elrepo.noarch.rpm
#升级最新内核
yum --enablerepo=elrepo-kernel install kernel-ml -y
```

2. 升级完毕后修改`/etc/grub.conf`将default=0修改为default=1（如果原来是1就改为0），然后`reboot`重启服务器。

3. 查看内核是否升级成功

   > 终端输入`uname -r`查看内核版本，大于4.9即成功

## 开启BBR

1. 直接复制以下命令即可：

   ```shell
   #修改配置
   cat >>/etc/sysctl.conf << EOF
   net.core.default_qdisc=fq
   net.ipv4.tcp_congestion_control=bbr
   EOF
   #使配置生效
   sysctl -p
   ```

2. 输入下面的命令来检测，如果看到返回的结果包含bbr 说明成功

   ```shell
   sysctl net.ipv4.tcp_available_congestion_control
   lsmod | grep bbr
   ```

## 设置防火墙

1. 编辑配置文件

   > `vim /etc/sysconfig/iptables`
   >
   > 在`-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT`这一行下面添加**shadowsocks**需要的端口号
   >
   > 格式为（如添加端口号8888）
   >
   > `-A INPUT -p tcp -m state --state NEW -m tcp --dport 8888 -j ACCEPT`

2. 重启防火墙

   > `service iptables restart`

## 部署shadowsocks

1. 安装shadowsocks

   ```shell
   yum -y update
   yum -y install vim python-pip
   pip install shadowsocks
   ```

2. 创建Shadowsocks的配置文件

   运行命令：

   > ```shell
   > mkdir /etc/shadowsocks
   > cd /etc/shadowsocks
   > vim ipv4.json
   > ```

   内容如下：

   > ```json
   > {
   >     "server":"your_server_ip",
   >     "server_port":8989,
   >     "local_address": "127.0.0.1",
   >     "local_port":1080,
   >     "password":"yourpassword",
   >     "timeout":600,
   >     "method":"aes-256-cfb",
   >     "fast_open": true,
   >     "workers": 1
   > }
   > ```

   > 各字段的含义：
   > server：服务器 IP (IPv4/IPv6)，注意这也将是服务端监听的 IP 地址
   > server_port：监听的服务器端口
   > local_address：本地监听的 IP 地址
   > local_port：本地端端口
   > password：用来加密的密码
   > timeout：超时时间（秒）
   > method：加密方法，可选择 “bf-cfb”, “aes-256-cfb”, “des-cfb”, “rc4”, 等等。默认是一种不安全的加密，推荐用 “aes-256-cfb”
   > fast_open：true 或 false。如果你的服务器 Linux 内核在3.7+，可以开启 fast_open 以降低延迟。开启方法：

   多端口内容如下：

   > ```json
   > {
   >     "server":"your_server_ip",
   >     "port_password":{
   >      "port":"password",
   >      "port":"password",
   >      "port":"password",
   >      "port":"password",
   >      "port":"password"
   >      },
   >     "local_address": "127.0.0.1",
   >     "local_port":1080,
   >     "timeout":600,
   >     "method":"aes-256-cfb",
   >     "fast_open": true,
   >     "workers": 1
   > }
   > ```

3. 开启`fast_open`

   `echo 3 > /proc/sys/net/ipv4/tcp_fastopen`

4. 测试启动

   > 运行`ssserver -c /etc/shadowsocks/ipv4.json start`
   >
   > 如没报错说明成功

5. 后台开机启动

   > 执行`vim /etc/rc.d/rc.local`
   >
   > 在`touch /var/lock/subsys/local`这一行上面添加
   >
   > `ssserver -c /etc/shadowsocks/ipv4.json -d start`
   >
   > 保存退出
   >
   > ps:如果要同时监听ipv4和ipv6，可以这样写
   >
   > ```shell
   > # 假如/etc/shadowsocks目录下有两个配置文件，一个ipv4，一个ipv6
   > ssserver -c /etc/shadowsocks/ipv4.json -d start --pid-file ss1.pid
   > ssserver -c /etc/shadowsocks/ipv6.json -d start --pid-file ss2.pid
   > ```

## 参考

1. [ssh修改登录端口禁止密码登录并免密登录](https://www.jianshu.com/p/b294e9da09ad)
2. [linux开启密码登陆](https://blog.csdn.net/ch2009120504/article/details/53170102)
3. [远程连接Linux，如何使程序断开连接后继续运行](https://blog.csdn.net/lyjcn/article/details/52780555)
4. [[日常折腾](三)配置SS同时监听IPv4/IPv6+多端口分享SS服务](https://blog.csdn.net/DesmondCobb/article/details/75600559)
5. [CentOS6.5安装部署Shadowsocks服务器](http://blog.51cto.com/zlyang/1891026)
6. [[CentOS 6/7升级最新内核并开启Google BBR](https://www.xiaoz.me/archives/9919)](https://www.xiaoz.me/archives/9919)
7. [解决SSH自动断线，无响应的问题。](https://www.coder4.com/archives/3751)
8. [Vultr 修改 Root 密码的方法](https://zhuanlan.zhihu.com/p/35779715)



