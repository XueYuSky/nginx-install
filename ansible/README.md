## 本Git代码适用范围
假设你有一个Web站点 http://dog.xmu.edu.cn ，IP地址为IPv4 1.2.3.4 ，你再提供一台配置了IPv6的Ubuntu 18.04 LTS服务器，Clone我的代码，跑一条命令，会帮你把HTTPS和HTTP/2全部配置完毕，然后你测试正常后，修改下DNS，把 dog.xmu.edu.cn 指向新的IPv4和IPv6地址即可。

中间是无缝的，干净的，测试完备的。时间在5分钟。

## 具体步骤
- 安装一台Ubuntu 18.04 LTS，配置好IPv6地址。
- Clone代码 https://github.com/haishanzheng/nginx-install 
- cp hosts.template hosts.real，配置你的服务器IP地址、控制机IP、域名、上游原始IP等信息
- 跑一下 ansible-playbook site.yml -i hosts.real --ask-become-pass，5分钟安装完毕
- 跑一下 certbot --nginx certonly ，申请一个免费的Lets Encrypt证书。
- 再跑下 ansible脚本，加入HTTPS支持。因为Ansible脚本是幂等的，所以你跑几千次都没问题。
- 跑下curl测试，强制域名指向新的IP地址和测试IPv6。curl --resolve dog.xmu.edu.cn:443:2001:da8:e800::42 -I --http2 https://dog.xmu.edu.cn -6 -v
- 如果curl正确，则更改DNS即可。再保险点，本地更改DNS，使用浏览器测试。

## 代码
代码fork自中科大张焕杰的 https://github.com/bg6cq/nginx-install ，PR暂时未提交，张焕杰老师的文档对Nginx进行了加固，对系统进行了配置优化，可做为一步步操作手册，了解内部的具体配置机制，张焕杰也带了个sh自动化部署脚本，依赖较少。ansible目录为我编写的自动化部署脚本，依赖Python3，可重复运行，幂等，会不定期同步张焕杰的配置。

## Ubuntu 18.04 LTS 系统安装

系统安装建议使用 Alternative Ubuntu Server installer。可以配置LVM等。

安装时请确保配置好IPv4、IPv6地址。选择自动更新安全补丁。配置好时区。

安装完成第一步请打开ufw，限制只有主控机可以访问22或者所有端口。命令为

```bash
sudo ufw enable
sudo default deny
sudo ufw allow from x.y.z.1
sudo ufw status verbose
```

目前只需配置这些，接下来会由Ansible自动配置防火墙打开HTTP、HTTPS端口。

如果你在Windows10下工作，建议打开Bash on Ubuntu on Windows。在里面跑Ansible控制远程服务器。

## 运行Ansible命令
ansible-playbook site.yml -i hosts.real --ask-become-pass
需要输入目标服务器sudo的密码


## HTTPS
HTTPS使用免费的Lets Encrypt。

运行

certbot --nginx certonly 

输入Email等信息。不要去修改Nginx配置文件。如果没有申请SSL证书，Ansible fqdn_has_ssl_certificate变量为False，这只会有HTTP服务，没有HTTPS。

## 测试

curl --resolve dog.xmu.edu.cn:443:210.34.0.42 -I --http2 https://dog.xmu.edu.cn -v
curl --resolve dog.xmu.edu.cn:443:2001:da8:e800::42 -I --http2 https://dog.xmu.edu.cn -6 -v

