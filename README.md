# ansibletest
Automation



ansible webservers -m shell -a 'sed -i "s/# User=/User=/g" /etc/systemd/system/v2ray.service;sed -i "s/# Group=/Group=/g" /etc/systemd/system/v2ray.service;chown -R v2ray:v2ray /var/log/v2ray;systemctl daemon-reload;systemctl restart v2ray;ps -ef |grep v2ray'

ansible webservers -m shell -a "ps aux | grep v2"
