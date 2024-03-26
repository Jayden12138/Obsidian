
记录碎片化的信息


```


// memos搭建



// debian

1. debian - docker安装
https://docs.docker.com/engine/install/debian/#install-using-the-repository

docker compose 安装
官方下载太慢了，直接从github release中下载了需要的版本，传了上去
https://github.com/docker/compose/releases

# sudo mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose

# chmod +x /usr/local/bin/docker-compose

# docker-compose --version

ps
这里选的版本依赖于现在的系统版本
根据官方的脚本
curl -L https://get.daocloud.io/docker/compose/releases/download/v2.1.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# uname -s
Linux
# uname -m
x86_64

=> docker-compose-linux-x86_64


2. nginx proxy manager
nginx 反向代理
https://github.com/NginxProxyManager/nginx-proxy-manager

安装步骤参考
https://blog.laoda.de/archives/nginxproxymanager#%E5%AE%89%E8%A3%85-Nginx-Proxy-Manager


3. memos
tip: 第一次拉取部署报错，这里需要先把对应端口打开

安装步骤参考
https://blog.laoda.de/archives/docker-install-memos#54-%E5%88%9B%E5%BB%BA%E5%AE%89%E8%A3%85%E7%9B%AE%E5%BD%95




nginx proxy manager
	http://101.43.168.133:81/
memos 
	http://101.43.168.133:5230/



```


- [ ] 反向代理
- [ ] 域名备案
- [ ] 微信bot





