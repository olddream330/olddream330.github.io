---
title: Python运维(高级篇)(Partial)
date: 2019-04-17 21:57:04
tags: Python运维
---


## 第二部分 高级篇

### pexpect
- expect的python封装， ssh、ftp、passwd、telnet自动交互
- spawn
  ```
  child1 = pexpect.spawn('ls', ['-latr', '/tmp'])

  shell_cmd = 'ls -l | grep LOG > logs.txt'
  child2 = pexpect.spawn('/bin/bash', ['-c', shell_cmd])
  child2.expect(pexpect.EOF)
  ```
- run
  ```
  pexpect.run('ls', ['-latr', '/tmp'])
  ```
- pxssh
  ```
  s = pxssh.pxssh()
  s.login(hostname, username, password)
  s.sendline('uptime')
  s.prompt()  # 匹配系统提示符
  s.before  # 系统提示符出现之前的命令输出
  s.logout()
  ```

### paramiko
```
paramiko.util.log_to_file()
ssh = paramiko.SSHClient()
ssh.load_system_host_keys()  # 默认~/.ssh/
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect(hostname, username, password)

stdin, stdout, stderr = ssh.exec_command('free -m')

ssh.close()
```
- SSHClinet
  - connect()
  - exec_command()
  - load_system_host_key()
  - set_missing_host_key_policy()
    - AutoAddPolicy
    - RejectPolicy
    - WarningPolicy
- SFTPClient
  - from_transport()
    ```
    t = paramiko.Transport(("192.168.1.102", 22))
    t.connect(username, password)
    sftp = paramiko.SFTPClient.from_transport(t)
    ```
  - put()
    - sftp.put(localpath, remotepath)
  - get()
    - sftp.get(remotepath, localpath)
  - mkdir()
  - remove()
  - rename()
  - stat()
  - listdir()

### Fabric
- fab
  ```
  fab -p password -H host1,host2 -- 'uname -s'
  ```
- fabfile
  - env  全局设定
- fabric.api
  - local()  本地
  - lcd()  本地
  - cd()  远程
  - run()  远程
  - sudo()  远程
  - put()  远程
  - get()  远程
  - prompt()  获得用户输入信息
  - confirm()  获得提示信息确认
  - reboot()  远程
  - @task()
  - @runs_once()

### Yorserver
- [Github](https://github.com/yorkoliu/pyauto/tree/master/%E7%AC%AC%E5%85%AB%E7%AB%A0)
- Python自带HTTP服务
  - BaseHTTPServer  基本的web服务
  - SimpleHTTPServer  GET, HEAD
  - CGIHTTPServer POST
- Yorserver
  - HTTP缓存
    - Expires  `Expires="7d"`
    - max-age
    - Last-Modified
  - HTTP压缩
    - gzip
  - HTTP SSL
    - privatekey
    - certificate
    ```
    # 生成RSA密钥
    openssl genrsa -des3 -out server.key 1024
    # 复制
    openssl -rsa -in server.key -out app.key
    # 生成证书
    openssl req -new -key server.key -out server.csr
    # 签发证书
    openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
    ```
  - 目录列表
  - 动态CGI

### Ansible
- YAML
  - 块序列描述
  - 块映射描述
- Installation
  - apt
    - /etc/apt/sources.list
      `deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main`
      ```
      sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
      sudo apt-get update
      sudo apt-get install ansible
      ```
  - pip
    ```
    sudo pip3 install -i https://mirrors.aliyun.com/pypi/simple ansible
    or
    pip install git+https://github.com/ansible/ansible.git@devel
    ```
- 配置
  - `/etc/ansible/hosts`添加主机IP同时定义到`[webservers]`组
  ```
  ansible 192.168.1.21 -m ping -k
  ansible webservers -m ping -k  # -k 要求提供root密码
  or
  ansible webservers -m ping -u ansible -sudo
  ```
- ssh无密码访问
  - 生成密钥： `ssh-keygen -t rsa`, id_rsa为私钥， id_rsa.pub为公钥， 需要下发到被控主机`.ssh`并重命名authorized_keys
  - 下发公钥： `ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.21`
  - 校验： `ssh -T root@192.168.1.21`
- 常用API
  - 远程
    ```
    ansible webservers -m command -a "free -m"
    ansible webservers -m script -a "/home/test.sh 12 34"  # 在远程执行主控端脚本
    ansible webservers -m shell -a "/home/test.sh"  # 执行远程脚本
    ```
  - copy
    ```
    ansible webservers -m copy -a "src=/home/test.sh dest=/tmp/ owner=root group=root     mode=0755"
    ```
  - stat
    ```
    ansible webservers -m stat -a "path=/etc/sysctl.conf"
    ```
  - get_url
    ```
    ansible webservers -m get_url -a "url=http://www.baidu.com dest=/tmp/index.html mode=0440 force=yes"
    ```
  - yum, apt
    ```
    ansible webservers -m apt -a "pkg=curl state=latest"
    ansible webservers -m yum -a "name=curl state=latest"
    ```
  - cron
    ```
    ansible webservers -m cron -a "name='check dirs' hour='5,2' job='ls -alh > /dev/null'"
    # 等同于
    * 5,2 * * * ls -alh > /dev/null salt '*' file.chown /etc/passwd root root
    ```
  - mount
    ```
    ansible webservers -m mount -a "name=/mnt/data src=/dev/sd0 fstype=ext3 opts=ro state=present"
    ```
  - service
    ```
    ansible webservers -m service -a "name=nginx state=stopped"
    ansible webservers -m service -a "name=nginx state=restarted"
    ansible webservers -m service -a "name=nginx state=reloaded"
    ```
  - sysctl
  - user
    ```
    # 添加用户johnd
    ansible webservers -m user -a "name=johnd comment='John Doe'"
    # 删除用户johnd
    ansible webservers -m user -a "name=johnd state=absent remove=yes"
    ```
- playbook


```python
import yaml

input_list = """
-
 - Hes
 - Pap
 - Apa
 - Epi
-
 - China
 - USA
 - Germany
"""

obj1 = yaml.load(input_list, Loader=yaml.FullLoader)
print("obj1: ", obj1)

input_dict = """
hero:
  hp: 34
  sp: 8
  level: 4
orc:
  hp: 12
  sp: 0
  level: 2
"""

obj2 = yaml.load(input_dict, Loader=yaml.FullLoader)
print("obj2: ", obj2)
```

    obj1:  [['Hes', 'Pap', 'Apa', 'Epi'], ['China', 'USA', 'Germany']]
    obj2:  {'hero': {'level': 4, 'sp': 8, 'hp': 34}, 'orc': {'level': 2, 'sp': 0, 'hp': 12}}

