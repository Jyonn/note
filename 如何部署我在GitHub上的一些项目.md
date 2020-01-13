# 如何部署我在GitHub上的一些项目
#### 工刀 最后一次更新 Oct. 3, 2018

涉及项目有：  
- `disk`浑天匣网络云盘后端  
- `unsplash`随机图片获取  
- `hiphop`冰山押韵  
- `saying`随机语录获取  
- `account`齐天簿单点登录系统  
- `enGIF`奇奇九号微信小程序  
- `multiss`多用户SS帐号获取系统  

这里边，有些项目需要依托七牛云的云存储方案（如`disk`，`account`和`enGIF`），有些项目需要前导项目部署完毕后才能部署（如`disk`和`multiss`需要`account`先部署完成，而`account`的前端部分有用到`saying`和`unsplash`项目的API），还有一些项目需要第三方网站的操作（如`unsplash`需要在[Unsplash高质量免费图片分享网站](https://unsplash.com)注册开发者帐号，`enGIF`的前端交互界面我注册了[微信小程序](https://mp.weixin.qq.com/cgi-bin/wx)开发者帐号）。  
此外，你需要一个域名（我选择的腾讯云，可以免费申请SSL证书，未来考虑使用`Let's Encrypt`），和一台服务器。

##一些假设
- 你的域名为`mydomain.com`，如果是在国内域名服务网站注册的域名，需要及时备案
- 你在本机安装了[phpMyAdmin](https://www.phpmyadmin.net/)或其他远程可视化控制数据库的工具（当然也可以不安装，如果你对数据库的操作了如指掌）

## 七牛云存储
`account`齐天簿单点登录系统需要存储用户头像，而`disk`浑天匣网络云盘需要存储用户上传的资源文件。我的服务器本身无法承载大型流量，因此将这些存储操作都交给七牛云；可以在前端直接上传到七牛云服务器，在后端接收七牛云反馈的文件信息。

注册用户后，在[密钥管理](https://portal.qiniu.com/user/key)界面创建密钥对。  
在[对象存储](https://portal.qiniu.com/bucket)界面新建两个存储空间，**一个公开，一个私密**。公开空间用于存储如`account`项目中用户头像等无需隐私的资源；私密空间用于存储如`disk`项目中用户存储在网络云盘里的资源文件。

接下来要配置两个存储空间的个性化域名。（为了确保连接安全性，建议使用HTTPS协议，超过一定流量后需要付费）。现假设公开空间的域名为`public.mydomain.com`，私密空间的域名为`private.mydomain.com`。在存储空间页面“点击绑定一个域名”，进行配置。

### 给存储空间配置HTTPS协议
申请SSL证书，我用的是腾讯云提供的[免费型DV版SSL证书](https://console.cloud.tencent.com/ssl/apply)，未来考虑使用`Let's Encrypt`。  
申请需要审核一段时间才能通过，下载得到`public.mydomain.com.zip`和`private.mydomain.com.zip`，解压之后只需要`nginx`文件夹中的两个文件。
在七牛云的[SSL证书服务](https://portal.qiniu.com/certificate/ssl)中点击上传自有证书，把`.crt`文件复制到在`证书内容`框中，把`.key`文件复制在`证书密钥`框中。
在绑定域名的配置页面中，填写加速域名（`public.mydomain.com`和`private.mydomain.com`），通信协议选择`HTTPS`，并选择相应的证书文件。

### 图片样式
添加以下两个图片样式。  

| 名称  | 处理接口  |
|------|----------|
| small | `imageView2/1/w/200/h/200/interlace/1/q/75|imageslim` |
| gif   | `imageMogr2/format/gif/blur/1x0/quality/75|imageslim` |

`small`一般用于对用户上传的头像进行压缩，而`gif`在`engif`项目中有用到。

## 云服务器
- 操作系统：Ubuntu 16.04LTS 及以上  
- 自行安装MySQL数据库（如有云数据库则跳过此条件）  
- 自行安装Nginx并卸载Apache

### 每个项目都有虚拟运行环境
```
sudo apt-get install libmysqlclient-dev, pip3
pip3 install virtualenv
```

### 每个项目在nginx的配置
nginx配置目录：`/etc/nginx/sites-enabled/default`  

#### django项目基本配置结构
PROJECT是此项目的名称，如`disk`，`account`等，为了配置HTTPS需要申请SSL证书；PROJECT_PORT是分配给每个Django项目的本地端口。
  
```nginx
# 80强制跳转443
server {
        listen 80;
        listen [::]:80;
        server_name PROJECT.mydomain.com;

        rewrite ^(.*)$ https://$host$1 permanent;
}
server {
        listen 443;
        listen [::]:443;
        server_name PROJECT.mydomain.com;
        ssl_certificate /path/to/1_PROJECT.mydomain.com_bundle.crt;
        ssl_certificate_key /path/to/2_PROJECT.mydomain.com.key;
        location / {
                proxy_pass http://localhost:PROJECT_PORT;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header token $http_token;
                proxy_set_header Authorization $http_Authorization;
        }
}
```

#### 普通前端项目基本配置结构

```
# 80强制跳转443
server {
        listen 80;
        listen [::]:80;
        server_name PROJECT.mydomain.com;

        rewrite ^(.*)$ https://$host$1 permanent;
}
server {
        listen 443;
        listen [::]:443;
        server_name PROJECT.mydomain.com;
        ssl_certificate /path/to/1_PROJECT.mydomain.com_bundle.crt;
        ssl_certificate_key /path/to/2_PROJECT.mydomain.com.key;
        root /path/to/PROJECT_ROOT;
        index index.html;
        try_files $uri $uri/ /index.html;
}
```
