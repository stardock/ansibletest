# ansibletest
Automation

我发现现在官方的一键安装脚本生成的systemd的启动文件，指定的是root用户运行的，毕竟v2ray是对外服务的，我担心会有人通过开放的端口可能存在的漏洞获得root权限，所以我做了个小小的设置，让v2ray以普通用户身份运行，减少攻击获取root权限的几率  
1、创建用于运行v2ray的用户，且不允许执行解释器  
`useradd v2ray -s /usr/sbin/nologin`  

2、修改v2ray.service文件，加入User和Group两项  
`vim /etc/systemd/system/v2ray.service`  

```
[Unit]
Description=V2Ray Service
After=network.target
Wants=network.target

[Service]
User=v2ray
Group=v2ray
Type=simple
PIDFile=/var/run/v2ray.pid
ExecStart=/usr/bin/v2ray/v2ray -config /etc/v2ray/config.json
Restart=on-failure

[Install]
WantedBy=multi-user.target
```  
执行systemctl daemon-reload  

3、为日志目录赋权  
``chown -R v2ray:v2ray /var/log/v2ray``  

4、重启服务  
`systemctl restart v2ray`  

5、查看运行状态  
`ps -ef |grep v2ray`  
```
v2ray     3382     1  0 21:49 ?        00:00:00 /usr/bin/v2ray/v2ray -config /etc/v2ray/config.json
root      3484  2073  0 22:04 pts/0    00:00:00 grep v2ray
```  
这样就可以看到v2ray是以普通用户运行的了   


那么问题来了，如何push到所有小鸡？
```
##批量更改v2启动模式
ansible webservers -m shell -a 'sed -i "s/# User=/User=/g" /etc/systemd/system/v2ray.service;sed -i "s/# Group=/Group=/g" /etc/systemd/system/v2ray.service;chown -R v2ray:v2ray /var/log/v2ray;systemctl daemon-reload;systemctl restart v2ray'
ansible webservers -m shell -a "ps aux | grep v2"

##批量删除v2日志
ansible webservers -m shell -a "rm -f /var/log/v2ray/error.log; touch /var/log/v2ray/error.log; chown -R v2ray:v2ray /var/log/v2ray"
```  

REF: https://github.com/v2ray/v2ray-core/issues/1011  
