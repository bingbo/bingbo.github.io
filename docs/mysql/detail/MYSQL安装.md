在mac装mysql直接用brew安装是比较方便的

__安装brew__

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

> 可参考http://brew.sh

__安装mysql__

```
brew install mysql
```

> 已安装在/usr/local/Cellar/mysql/5.7.13目录下

__建立连接__

```
brew link mysql
```

> 如果出现`Linking /usr/local/Cellar/node/6.2.2...  Error: Could not symlink include/node /usr/local/include is not writable.`的错误，请先执行`sudo chown -R 'whoami' /usr/local/include`解决权限问题

__启动mysql__

```
brew services start mysql
```

> 该方式是在后台运行服务

__前台方式启动mysql__

```
mysql.server start
```

__连接mysql__

```
mysql -uroot
```

