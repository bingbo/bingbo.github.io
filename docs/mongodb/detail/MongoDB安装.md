
在mac通过Homebrew进行安装，当然你得先安装Homebrew，有关Homebrew可以参考这里：http://brew.sh。   同时也可以手动自己安装。

官方参考文档：https://docs.mongodb.org/manual/installation/

## 通过Homebrew安装

1. 更新brew包数据

```bash
brew update
```

2. 安装MongoDB

```bash
brew install mongodb

brew install mongodb --with-openssl

brew install mongodb --devel
```

## 手动安装

1. 下载二进制包

```bash
curl -O https://fastdl.mongodb.org/osx/mongodb-osx-x86_64-3.2.3.tgz
```

2. 解压下载的文件包

```bash
tar -zxvf mongodb-osx-x86_64-3.2.3.tgz
```

3. copy文件到目标目录，即安装目录

```bash
mkdir -p mongodb
cp -R -n mongodb-osx-x86_64-3.2.3/ mongodb
```

4. 配置环境变量

```bash
export PATH=<mongodb-install-directory>/bin:$PATH
```

## 运行MongoDB

1. 创建数据目录

```bash
mkdir -p /data/db
```

2. 设置数据目录的访问权限

```bash
chmod -R 775 /data/db
```

3. 运行

```bash
mongod --dbpath /data/db
```

4. 使用mongodb客户端命令行

```bash
mongo
```