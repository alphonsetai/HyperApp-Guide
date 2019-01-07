# FAQ

## 索引
* HyperApp 问题
    * 关于国区被下架的解决方法
    * 如何申请 TF
    * 如何升级应用
    * 常见安装错误
    * 服务器错误
* 应用问题
    * 文件上传大小限制
    * nextcloud安装问题
    * wordpress 需要提供ftp凭据
    * 网页无法访问 (502,503)
* 爱国问题
    * 手机爱国电脑不爱国
    * 无法爱国的问题
* 系统问题
    * 如何手动开启 BBR
* [GCP 问题](#gcp)
    * [结算账号被停用](proxy/get-started.md)
    * [密钥认证失败](proxy/get-started.md)


## HyperApp 常见问题

### 关于国区被下架的问题

1. 「换区」临时的下载&更新方案是将自己账号换区，比如加拿大区（不需要绑定支付方式），然后点击购买即可（不需要再次付费）。
2. 「TestFlight」在国区购买的用户可以将「Apple 提供的收据」（必需）和邮箱发给我加入 TF，不换区也能一直保持更新。邮件发送至 HyperAppCloud🌀gmail.com
3. iMazing 免费版就可以安装已下架的应用，是目前最简单的办法，搜索即可。


### 如何申请 TF

在 `App` 里面点击 `发送反馈邮件` 将主题改为 `申请 TF` 并且附上购买收据即可。


### 如何升级应用？

1. 在服务器下面的 Docker 监控里选择你的应用，轻点后选择 `Pull Image` 即可更新镜像，然后在应用那里 `更新配置`
2. 手动的话可以 `docker ps` 找到应用的镜像名字，然后 `docker pull 镜像`，然后在应用那里 `更新配置`


### 安装错误


#### 安装应用时报错 "i686 not recognized platform"

* 系统必须是64位(X86-64)Linux系统
* 虚拟化方式不支持 OpenVZ


#### Bind for *** failed:port is already allocated.

端口重复了，不同的应用、进程不能使用同一个端口


#### 为什么在安装应用的时候.会卡在安装中非常久(这种情况多出现在国内VPS中)？

请先确定你的VPS可以正确联网再参考下列步骤:
* 方法1: 在hyperapp终端左下角第一个按钮中.选择安装阿里云国内Docker源
* 方法2: 阿里云用户请参考[使用阿里云镜像加速服务](Aliyun-docker.md)
* 终极解决办法: 远离国内VPS.
* 如果你使用的国外 VPS 依然非常慢，那估计是你VPS的网络比较慢，有些应用的镜像很大，下载需要时间。


#### 提示连接失败/无法连接

1. 请不要打开VPN连接hyperapp.（或者在对应的App中加一条规则，设置你的服务器IP直连）
2. 检查服务器IP是否变了（GCP 如果没有用静态IP，服务器关机后再重启IP可能会变）
3. 检查SSH端口有没有填对（默认22）
4. 也许你的IP被墙了


#### 提示 transport read 或者 timeout

1. 不要开 VPN 使用 HyperApp，并且进行操作的时候不要切换应用.
2. timeout 可能是掉线了，刷新下服务器状态，看看应用状态是否改变。


#### Conflict

`docker: Error response from daemon: Conflict. The container name *** is already used by container ....`

应用已经安装过了，更新了配置要点击“更新配置”


#### docker command not found

1. 确保你 VPS 的操作系统支持 Docker。
2. HyperApp 会自动安装 Docker，出现这类问题可能是 Docker 安装失败，你可以手动执行 `curl -fsSL get.docker.com | sh` 安装最新docker


#### cannot connect to the Docker daemon

1. 请确保你的 Linux 版本符合要求
2. 请确保一定要先安装 BBR 再安装应用，如果你不幸搞错了顺序，参考下一条
3. 尝试在终端的快捷命令中点击 `Start Docker` 手动启动下 Docker 或者启动任意一个应用


#### 如果先安装了docker再安装bbr导致无法启动docker怎么办?

执行 `rm -rf /var/lib/docker/aufs` 清除docker网络配置.然后重新安装所有应用即可.


#### sudo: no tty present and no askpass program specified

你需要修改 `sudo` 配置文件允许当前用户无密码运行 `sudo`


#### 操作Docker时提示权限不足(Permission Denied)

原因：当前用户没有docker权限，加到docker组才有，执行命令


```
sudo usermod -aG docker $USER
```

#### 安装应用时安装失败提示"overlay"已满

原因：已安装或者曾经安装过的应用太多，因为Docker自身的机制，删除容器并不会删除镜像，久而久之硬盘可能会被塞满

首先进入命令行，输入以下命令来查询已存在的镜像

```
docker image ls
```
然后找到不需要的或者占用体积特别大的镜像，复制它的"REPOSITORY"或者"IMAGE ID",然后键入以下命令来删除镜像

```
docker image rm 镜像ID或者REPOSITORY
```
删除完成之后可看到硬盘空间已经释放，可以继续安装想要的应用

#### 使用bbr安装脚本的时候提示"Inappropriate ioctl for device"
使用键盘随便输入几个字符并且回车两下.原因是bbr安装脚本需要等待输入来执行下一步


##### 部署 Nginx SSL Support 报错提示找不到/不存在Nginx Proxy 但 Nginx 已经正确安装

* 如果将同一个应用部署到多个服务器，最好在安装时一次选择多个服务器而不要开启多个应用。

检查 SSL Support 是否在同一机器有对应的Nginx


---


## 应用问题

### 网页无法访问

常见原因：

1. 如果你启用了 `HTTPS` 请稍等片刻，SSL Support 获取证书需要时间（大约几分钟）。
2. 有些应用安装后也需要几分钟的初始化时间（比如数据库初始化），请稍等几分钟再试。


如果错误继续，参考下面过程排错：

1. 刷新服务器状态，确保应用处于 `running` 状态。
2. 如果不在运行中的状态，说明安装后的初始化没有成功，查看应用日志可以看到出错原因。
3. 如果你要寻求帮助，请发出此处的日志截图，且要贴全。

### 关于 Nginx 的上传限制

如果你使用了 `Nginx Proxy`，则默认有个2M的上传限制，你可以新建一个文件 `/srv/docker/nginx/vhost.d/default` 添加一行  `client_max_body_size 100m;` 来解决这个问题（一般情况下 Nginx 会自动重启，如果没有则需要手动重启下 Nginx Proxy）

你也可以直接运行下面的命令，会自动创建上面的所说的文件:

```sh
echo "client_max_body_size 100m;" > /srv/docker/nginx/vhost.d/default
```
如果原方案不行，尝试以下两种方案：
1. ```echo "client_max_body_size 100m;" | sudo tee -a /srv/docker/nginx/vhost.d/default```
2. ```sudo su echo "client_max_body_size 100m;" > /srv/docker/nginx/vhost.d/default```

以上任选一种执行后，手动重启nginx

**除了 Nginx 外，PHP 还可以有自己的上传限制，请参考 https://github.com/waylybaye/HyperApp-Guide/issues/152**


### nextcloud 安装问题

1. 设置数据库时,填入数据库的账号密码之后,提示authentication出现问题,无法认证
   这是MySQL的新版本更新了验证方式,比较简单的方式是在安装mysql时,手动选择`5.x`版本的镜像,不要选择`latest`,另一个方案是使用Mariadb,两者在后续操作上没有区别

2. 进行安全检查时,提示没有设置`referer`
   解决方法参考更改nginx的默认上传大小,在``` srv/docker/nginx/vhost.d/```目录下,新建一个文件,文件名为你为nextcloud分配的域名(要完整),在里面加入两条记录```add_header Referrer-Policy no-referrer;client_max_body_size 500m;``` ,你可以使用 ```echo "add_header Referrer-Policy no-referrer;client_max_body_size 500m;" >> /srv/docker/nginx/vhost.d/你的域名```
   
### wordpress 提示设置ftp

在自动更新wordpress新版本时,提示填入ftp凭据,这是由于在配置wordpress的过程中文件没有写权限造成的,所以需要重新更改权限.查询一下wordpress的container name,然后执行 ```docker exec CONTAINER_NAME chown -R www-data:www-data /var/www/html```,`CONTAINER_NAME`填写你查询到的结果,具体参考[这里](https://github.com/docker-library/wordpress/issues/24).

## 爱国问题

### 手机可以电脑不行：
请先查看客户端日志，并贴出来用全局模式测非Chrome、Firefox浏览器（因为Chrome浏览器的很多插件会改代理设置）如果全局可以就能断定是pac规则或者浏览器的原因。


### 无法科学上网的问题

#### 排查服务端的问题

1. HyperApp 里面刷新一下服务器状态，看应用是否启动成功(显示 up xxx seconds/minutes/hours)
2. 点击 `查看日志` 看看应用日志中是否有错误提示
    * ss-libev 如果有一些连接错误的信心，一般是客户端的密码、加密方式、混淆没填对。
3. 如果上两步都没有问题，请检查
    * 是否是机器防火墙问题（参考下面的第二项）
    * 是否是客户端配置错误（参考下面的客户端排错）

#### 排查客户端的问题

* 是否使用了跟服务端一致的配置。建议通过扫描二维码的方式添加。
* 如果你用 Wingy 不要开 OBFS （除非你自己足够了解）
* 另外不要用 SSR 连 SS，也不要用 SS 连 SSR
* ss-libev 客户端的 http/tls 混淆必须要和服务端一致。（服务器如果设置的 tls 混淆，客户端也必须用 tls）
* 最后如果你的问题不在上述思路中.请仔细查看文章中涵盖的hyperapp常见问题以及解决方案.实在搞不懂再上群提问并且请注意你的语气.没有人有义务帮助你.

### 如何排查服务器是否有防火墙阻拦了某个端口？

1. 在 VPS 上运行: `telnet localhost PORT(要检查的端口)` 能通说明服务端没问题。不通说明服务端启动失败。
2. 在电脑上运行`telnet IP PORT(要检查的端口)` 能通说明防火墙没问题。不通说明是防火墙的问题。
3. 在 VPS 上运行 `lsof -i :端口` 来查看是否有进程正在监听该端口，如果有说明是防火墙屏蔽了端口访问，如果没有说明部署失败。


## 防火墙设置教程

* 临时禁用系统防火墙
  * CentOS7: service firewalld stop
  * CentOS6: service iptables stop
  * Ubuntu16: service ufw stop
* 永久禁用系统防火墙
  * CentOS7: systemctl disable firewalld
  * Ubuntu16: systemctl disable ufw
* 系统防火墙永久添加端口
  * CentOS7: firewall-cmd --permanent --zone=public --add-port=端口/tcp
* 平台防火墙
  * GCE: 网络 → 防火墙规则
    * 点击`创建防火墙规则`
      * 名称：随便输入一个名称
      * 目标：选择 `网络中的所有示例`
      * 来源过滤：`0.0.0.0/0`
      * 协议和端口：`指定的协议和端口` 下面输入 `tcp;udp:端口号`
  * AWS: EC2 控制面板 → 安全组
  * 阿里云: 云服务器 ECS → 安全组

----


## 系统问题

### 如何手动开启 BBR

注意以下命令需要 `root` 权限，非 root 用户先执行 `sudo su -` 切换到 root 账户

1. 运行 `uname -r` 看看是不是内核 >= 4.9，不是的话请先升级内核
2. 执行 `lsmod | grep bbr`，查看 BBR 模块是否启用，如果结果中没有 tcp_bbr 的话就先执行
    ```sh
    modprobe tcp_bbr
    echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
    ```
3. 执行下命令开启设置 TCP 使用 BBR 发包
    ```sh
    echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
    echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
    ```
4. 保存生效 `sysctl -p`


### 验证 BBR 是否已经启用

```sh
sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
```

如果结果都有bbr, 则证明你的内核已开启bbr

----

## GCP 问题

### 二维码显示不全，如何手动添加 GCP ？

1. 二维码上面有 `IP`，`用户名`，`端口` 信息，在 `HyperApp → 服务器` 中点击添加按钮，选择手动配置。
2. 在出现的窗口中，分别输入上面对应的字段，密码不填写，在最下面的密钥那里选择一个密钥（应该只有一个）。
3. 添加即可。但是稍后的可能会出现密钥认证失败的错误，所以建议将密钥名字改成你的 gmail 邮箱名字。或者按照教程，修改两处用户名一致即可。

## 厂商常见问题：

* 搬瓦工只支持 KVM虚拟化 的机器。
* Vultr/DigitalOcean/AWS 全都支持
* GCP没有领到试用金：重新打开试用页面再点击一次“Try it free”
* 50Kvm 的机器第一次登录需要记下下次登录的 SSH 端口
* DigitalOcean 的机器第一次登录需要改密码


----

## 其它常见问题:

###  关于squid内存不足无法正确启动的解决办法:

1. Ubuntu使用如下命令创建swap:

```bash
sudo su
fallocate -l 1G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
cp /etc/fstab /etc/fstab.bak
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
echo "vm.swappiness = 10" » /etc/sysctl.conf
sysctl -p
```

2. Centos请将第一个echo换成
  `echo "/swapfile swap swap defaults 0 0" | sudo tee -a /etc/fstab`

### V2ray不能连接排查

针对教程：V2Ray完美混淆教程（内含全平台客户端配置教程）

* 确定完全按教程操作
  * 填写 config.json新建docker默认值被遮挡，且与教程所给值不一致，需更改为srv/docker/etc/v2ray.json
  * 应用端口需手动填写，不要和别的教程一样默认自动抓取了
  * 域名填写低调一点的
  * Ubutu不用看这条，centos7注意开启防火墙，添加新端口（你填的port），并重启防火墙，下面写为yyy，在ssh中执行
  ```
  firewall-cmd --add-port=yyy/tcp --permanent && firewall-cmd --reload
  ```
  * 更新配置或安装后启动docker，注意要手动重启Nginx proxy和Nginx SSL Support

* 出错后检查思路
  * 先检查证书是否存在.tls域名是否正常.伪装应用是否正常
  * 再检查后端v2ray的服务端配置
  * 再检查客户端配置


### 关于V2Ray访问域名出现Bad Request的解决办法
  如果你按照教程搭建了`Nginx(tls)+V2Ray(websocket)`.那么访问证书域名的时候往往会出现bad request的情况.这是成功的标志.但是总不那么好看.这里给出一个跳转到其他域名的办法.执行以下命令即可.注意修改命令中的域名

```sh
echo -e "proxy_intercept_errors on;\nerror_page 400 = https://要跳转到的域名;" > /srv/docker/nginx/vhost.d/default
```
