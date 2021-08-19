# 给非root账号添加docker权限

```shell
#新建用户
adduser ${username}

#给用户添加密码
passwd ${username}

#将系统用户添加到docker用户组
sudo usermod -aG docker username

#查看docker用户组
cat /etc/group | grep docker
```

