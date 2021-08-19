```shell
# 查看所有运行的端口
netstat -ano

# 查看被占用端口对应的PID
netstat -aon|findstr "8080"

# 查看指定PID的进程
tasklist|findstr "9088"

# 结束进程
taskkill /T /F /PID 9088 

{
  "account":"admin",
  "password":"root",
  "username":"admin",
  "phone":"12456623435",
  "id":2
}
```

