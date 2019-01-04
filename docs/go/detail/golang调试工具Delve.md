### 安装Devle

```
go get -u github.com/derekparker/delve/cmd/dlv
```

### 调试

```
##开启调试
dlv debug main.go

##查看帮助
(dlv) help

##打断点
(dlv)b /Users/zhangbingbing/mygo/src/gin-app/service/user_service.go:13
##中间可通过再次打断点
(dlv) b dao.GetUsers

##然后运行c 来运行到断点
(dlv)c

##请求访问服务
curl localhost:8000/api/user/list

##即可看到断点情况
> gin-app/service.GetUsers() /Users/zhangbingbing/mygo/src/gin-app/service/user_service.go:13 (hits goroutine(7):1 total:1) (PC: 0x1625893)
     8: func init() {}
     9: 
    10: /**
    11: 获取用户列表
    12: */
=>  13: func GetUsers() (users []model.User, err error) {
    14:         users, err = dao.GetUsers()
    15:         //users, err = dao.GetUserV2()
    16:         return users, err
    17: }
    18: 

```

### 使用Delve附加到运行的golang服务进行调试

> 先编译并启动服务

```
go build main.go
./main
```

> 查找服务进程号

```
ps aux|grep main
```

> 然后附加操作

```
$dlv attach 23882
Type 'help' for list of commands.
(dlv) b /Users/zhangbingbing/mygo/src/gin-app/service/user_service.go:13
Breakpoint 1 set at 0x1625893 for gin-app/service.GetUsers() /Users/zhangbingbing/mygo/src/gin-app/service/user_service.go:13
(dlv) c
> gin-app/service.GetUsers() /Users/zhangbingbing/mygo/src/gin-app/service/user_service.go:13 (hits goroutine(7):1 total:1) (PC: 0x1625893)
     8: func init() {}
     9: 
    10: /**
    11: 获取用户列表
    12: */
=>  13: func GetUsers() (users []model.User, err error) {
    14:         users, err = dao.GetUsers()
    15:         //users, err = dao.GetUserV2()
    16:         return users, err
    17: }
    18: 
(dlv) n
```

### dlv常用命令

* 输入 b 打断点，
* 输入 n 回车，单步执行，
* 输入 print（别名p）输出变量信息　　
* 输入 args 打印出所有的方法参数信息
* 输入 locals 打印所有的本地变量

