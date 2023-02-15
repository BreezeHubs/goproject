### go 升级到1.19
```shell
wget https://golang.google.cn/dl/go1.19.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.19.linux-amd64.tar.gz
go version
```

### go work
项目集合的根目录运行
```shell
go work init
```
&nbsp;

### 安装protoc
```shell
wget https://github.com/protocolbuffers/protobuf/releases/download/v21.9/protobuf-cpp-3.21.9.tar.gz
tar -xvzf protobuf-cpp-3.21.9.tar.gz
cd protobuf-3.21.9/
./autogen.sh
./configure
make
sudo make install
protoc --version # 查看 protoc 版本，成功输出版本号，说明安装成功
```

> 提示：这里我们安装的 protoc 版本是 3.21.9。如果你安装了其他版本，后续执行 protoc 命令报错，可能需要你根据所安装的版本进行命令参数适配

在执行 autogen.sh 脚本时，如果遇到以下错误：
```
configure.ac:48: installing './install-sh'
configure.ac:109: error: required file  './ltmain.sh'  not found configure.ac:48: installing './missing'
benchmarks/Makefile.am: installing './depcomp'
conformance/Makefile.am:362: warning: shell python --version 2>&1: non-POSIX variable name
conformance/Makefile.am:362: (probably a GNU make extension)
conformance/Makefile.am:372: warning: shell python --version 2>&1: non-POSIX variable name
conformance/Makefile.am:372: (probably a GNU make extension)
parallel-tests: installing './test-driver'
autoreconf: automake failed with exit status: 1
```
可以通过以下命令配置 libtoolize，并再次运行 autogen.sh 来解决：
```shell
libtoolize --automake --copy --debug –force
./autogen.sh
```
&nbsp;

### 安装protoc-gen-go
```shell
go install github.com/golang/protobuf/protoc-gen-go@latest
```

### 安装miniblog
```shell
#https://github.com/marmotedu/miniblog/tree/master

git clone https://github.com/marmotedu/miniblog.git #master分支
go work use ./miniblog # 将 miniblog 项目添加到当前 Go 工作区中
```

### MariaDB 安装和配置
```shell
docker pull mariadb
mkdir -p $PWD/data/mariadb/data
docker run --name mariadb -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mariadb
```

登录数据库并创建 miniblog 用户
```shell
mysql -h127.0.0.1 -P3306 -uroot -p'123456' # 连接 MariaDB，-h 指定主机，-P 指定监听端口，-u 指定登录用户，-p 指定登录密码
```

创建 miniblog 数据库
```shell
cd ./miniblog

mysql -h127.0.0.1 -P3306 -uroot -p'123456'

MariaDB [(none)]> source configs/miniblog.sql; 
MariaDB [miniblog]> use miniblog ; 
Database changed
MariaDB [miniblog]> show tables; 
+--------------------+
| Tables_in_miniblog |
+--------------------+
| post               |
| user               |
+--------------------+
2 rows in set (0.000 sec)
```

### 安装部署 miniblog 服务
```shell
cd ./miniblog

sudo mkdir -p /data/miniblog /opt/miniblog/bin /etc/miniblog /var/log/miniblog # 创建需要的目录
make build # 编译源码生成 miniblog 二进制文件
sudo cp _output/platforms/linux/amd64/miniblog /opt/miniblog/bin # 安装二进制文件
sed 's/.\/_output/\/etc\/miniblog/g' configs/miniblog.yaml > miniblog.sed.yaml # 替换 CA 文件路径
sudo cp miniblog.sed.yaml /etc/miniblog/miniblog.yaml # 安装配置文件                      
make ca # 创建 CA 文件
sudo cp -a _output/cert/ /etc/miniblog/ # 将 CA 文件复制到 miniblog 配置文件目录
```

通过 Systemd 启动 miniblog 服务。配置命令如下：
```shell
sudo cp init/miniblog.service /etc/systemd/system/miniblog.service # 创建 Systemd Unit 文件
sudo systemctl daemon-reload # 重新加载 Systemd Unit 文件
sudo systemctl restart miniblog # 启动 miniblog 服务
sudo systemctl enable miniblog # 设置 miniblog 服务开机启动
```

测试 miniblog 服务是否成功安装
```shell
curl http://127.0.0.1:8080/healthz #测试 miniblog 服务是否成功安装
  {"status":"ok"}
```