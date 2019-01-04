### iterm安装

1. 下载安装：https://github.com/bingbo/blog/tree/master/tools/mac/iterm
2. 官网下载：http://iterm2.com/downloads.html

### 设置共享ssh会话

每天只需要输入一次token即可，编辑`~/.ssh/config`加入以下内容

```bash
Host *
    ControlMaster auto
    ControlPath ~/.ssh/master-%r@%h:%p
```

### iterm配色

1. 下载配色文件：https://github.com/bingbo/blog/blob/master/tools/mac/iterm/iterm2_wjl.itermcolors
2. 在iterm->preferences->profiles->default->colors->import导入配色文件
3. 选择iterm2_wjl配色文件

### iterm下使用rz/sz

1. 安装zssh

    ```bash
    brew install zssh
    ```

1. 然后用zssh xxx@xxx.com登录服务器
1. 上传文件：登录到测试机的时候，按Control+2，然后zssh会切换到本地目录，通过pwd和cd切换到对应的目录，sz -be ${filename} 传文件到测试机上的相应目录
1. 下载文件：在测试机上先执行 sz -be ${filename} ，然后看到一堆乱码，再 Control+2 ，zssh会切换到本地目录，通过pwd和cd切换到接受文件的目录，然后执行 rz -be

### Samba-本地与服务器传文件

1. 在测试机上安装samba

    ```bash
    sudo yum install samba
    sudo yum install samba-swat
    ```

1. 配置samba

    * 打开/etc/samba/smb.conf文件，并编辑
        
        ```bash
        在global的设置中，编辑 security = user
        在配置文件最后，添加以下内容：
        [myShare]
        path = /home/work
        valid users = work
        public = yes
        guest ok = yes
        writable = yes
        write list = work
        ```

    * 注意，上面的配置文件中所写work，必须为测试机上实际存在的账户名
    * 添加samba的账户访问权限

        ```bash
        在终端中执行以下命令：sudo pdbedit -a work
        然后设置登录密码
        ```

    > 注意，这里的work必须与上面保持一致

1. 启动samba

    ```bash
    sudo /etc/init.d/smb start
    ```

1. mac本地访问

    > 打开finder——前往——连接服务器——输入：smb://cq01-vm001(你的机器名)
    以后就能直接在此文件夹中访问，并编辑文件啦
