# nginx 配置mp4 播放

## 先决条件

1. 服务器操作系统是Centos 7
2. 用客户端工具连接 服务器, 连接方法之一可参考此文:`https://blog.csdn.net/ZZY1078689276/article/details/77280814`
3. 按下文操作
## 部署过程
### 服务器开机

```
正常开机
```

### 输入账号, 密码登录



![ff72d71c32485f2c94cc985077d25bc3.png](../_resources/cd3118bb2d5e45b098cb77813db8dd95.png)



#### 登录成功的样子



![168a09d71797ab24dbdd1cef9e23fc3d.png](../_resources/c151017823294350b921951299d84317.png)

![image-20200311101836119](/Users/xbs/Library/Application Support/typora-user-images/image-20200311101836119.png)

### 查看ip



![3a77b9bc02bd81088b79753f50bb893a.png](../_resources/9b9019e268e54be4aea492e9ac120e07.png)



找到自己能够用`工具` 连接的ip (内网或者公网)

### 使用SSH 工具(为了方便复制粘贴)

SSH 工具, 自行选择, 这里使用 xshell 做示范



![653071facea6abf2056792da7c729dfd.png](../_resources/cf5eb209966241d0babb83a0a50a83f3.png)



1. 点击箭头所指的位置



![b74c4cf000d9ddfbac7804be44715a64.png](../_resources/ca3879b7dd28450b90d92ee2c6f17cf8.png)



2. 依次按上图填入自己的信息


![23dbf82ce353080620a9244230fee5cc.png](../_resources/b7659efae47e4f019e2c99c58f529921.png)



3. 双击打开



![335af3bd626ef8a0212d8d2892f59370.png](../_resources/fa9cd70faa014171a1c20c8cd7cd77fd.png)


4. 登录成功样子

### 优化打开文件限制

```
echo "* soft nofile 65535" >> /etc/security/limits.conf 
echo "* hard nofile 65535" >> /etc/security/limits.conf
```



![1fc9f66b0346ade4d5d8f65c750f1646.png](../_resources/cdc4e927ebd444f5af24b54fbabcba7a.png)



### 关闭防火墙和`selinux`

```
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```



![1947bcbff9c4293fbbc0ebf8fbf3fcea.png](../_resources/f07d1d0c308d47928c3689990500dafa.png)



### 安装 `yum-utils`

```
# 此步骤需要确认服务器能够访问互联网, 并有有效的yum源, 默认是可以正常使用的
sudo yum install yum-utils -y
```


![ccd4fb16e98fed28502aa2612072be50.png](../_resources/74943b213a2f4fe29128fa3d0ce1272e.png)




### 安装nginx稳定版官方源

```
cat > /etc/yum.repos.d/nginx.repo << "EOF"
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
EOF
```



![b17dace6f35bd098955d66aa2592e0bd.png](../_resources/e18ff6ae934c44c99328f361bbe2724e.png)



### 安装nginx 

```
yum install -y nginx  # 这一步会比较慢, 只要不断就耐心等待
```



![2943224d22b795195e5256060c9b15e3.png](../_resources/8fbfed7032dc435e98b2dfaa71d245d0.png)



### nginx.conf 配置参考

```
\cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.baktest
cat > /etc/nginx/nginx.conf << "EOF"
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  10240;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
EOF
```



![4c706eb3c0ff1246a055a104435a9c46.png](../_resources/308694a2e2bd4c5c9f13f6131c715148.png)



### 视频网站配置文件参考

```
\cp /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.baktest
cat > /etc/nginx/conf.d/default.conf << "EOF"
server {
    listen       80;
    server_name demo.videodrm.cn;
    
    location / {
        
        charset utf-8;   # 支持中文
        autoindex on;  # 开启自动索引
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
EOF
```



![05d7a654f1a9f44c8687c0d6cbb6aee6.png](../_resources/03e119f297e24f80ba231f23310bc85d.png)


### 重启nginx

```
systemctl restart nginx
```



![32b9f5994ad1975511fedcfc26054d93.png](../_resources/f9149c7d46734d1fb811b7cc71f61449.png)



没有报错, 就可以继续下一步

#### 步骤解释

1. 将视频放入nginx 的 `root` 目录下,此示例为`/usr/share/nginx/html` 即可
2. 另外优化可以根据业务需要进行优化



### 删除`/usr/share/nginx/html`下的index文件

*** 此步骤出删除默认索引页, 所以确定自己没有网站*** 

 ```
rm -f /usr/share/nginx/html/index.html
 ```



![dcacbdd311220027c2451ed3582396f4.png](../_resources/09432ad2c529466dbd3b589636d89f51.png)



### 在`/usr/share/nginx/html`创建视频目录

```
mkdir /usr/share/nginx/html/语文
mkdir /usr/share/nginx/html/数学
mkdir /usr/share/nginx/html/英语
```



![4cae4cd3db92494d97048531455e3431.png](../_resources/2e054d65b6004ba8a191628c80b1a982.png)



### 上传视频到对应的目录, 例如上传视频到 语文目录下



![62427cccd22cd9e64b6d222509e9419e.png](../_resources/8cbe878c3dda48e3a844a334b0aa5dc5.png)



1. xshell 中切换到 语文目录下
2. 在xshell 中直接调用xftp 程序, 如果软件版本号相同, 会自动切换到语文目录下



![d4c7a4367fb5a819efb8aa25e7231237.png](../_resources/f07094f279a84f0aba4347c24549b54f.png)



选中视频, 右键传输



![a433cbe801aedbcfbfda6e7dab9b3a9c.png](../_resources/26f2a2c4bae84ce0a48ebf584fc19907.png)


传输成功



![a18f916c8c93f61a0f8688c141dd8d66.png](../_resources/fec995c112de47d1ad0a9960a0220cc9.png)



### 浏览器打开网址(ip), 例如:192.168.120.131


### 播放视频

选择刚刚上传到语文目录下的视频播放



![04a71f619bf2a780cfe99024a1d5587c.png](../_resources/704c3b39f3b848dbbad599f284f9c7bd.png)

