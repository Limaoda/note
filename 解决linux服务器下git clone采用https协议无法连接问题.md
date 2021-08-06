### 解决linux服务器下git clone采用https协议无法连接问题



##### 无法连接原因

端口22为ssh'默认端口，服务器对其它服务器的访问默认端口也是22端口，因此需要将端口修改为443（https默认端口）



##### 解决方案

```shell
#进入到~目录下面的.ssh下面
cd ~/.ssh/

#修改ssh配置，新建config文件 (vim config命令，有则编辑，无则创建)
vim config
```

在config文件中添加如下代码

```txt
Host github.com    /*服务器地址为github地址*/

User "HuQiaoDong"     /*github上的注册邮箱 为用户账号*/

Hostname ssh.github.com   /*服务器地址为github地址*/

PreferredAuthentications publickey  /*采用公匙*/

IdentityFile ~/.ssh/id_rsa    /*公匙文件路径*/

Port 443                           /*修改端口为443*/
```

##### 保存文件，重新运行git clone(https)，问题解决

