# Linux 安装部署

## JDK 安装部署



## Tomcat 安装部署

### 解压配置安装

1.  进入到 `/opt/software` 目录，然后解压文件

    ```bash
    tar -zxvf <压缩文件> -C <目标目录>
    ```

2.  配置环境变量

    ```bash
    # Java的环境变量
    export JAVA_HOME=<Java 目录>
    export PATH=$PATH:$JAVA_HOME/bin
    
    # tomcat的环境变量
    export CATALINA_HOME=<tomcat 目录>
    export PATH=$PATH:$CATALINA_HOME/bin
    ```

    配置完成后 `source /etc/profile` 刷新即可。

### 配置远程连接

开放Linux端口 8080

```bash
firewall-cmd --zone=public --add-port=8080/tcp --permanent
```



## MySql 安装部署

### 通过yum指令安装

1.  确认是否有wget指令，如果没有，则安装之

    ```bash
    yum install -y wget
    ```

2.  安装yum所需组件

    下载yum源

    ```bash
    wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
    ```

    安装yum源

    ```bash
    rpm -ivh mysql57-community-release-el7-11.noarch.rpm
    ```

3.  通过yum安装mysql

    ```bash
    yum install -y mysql-community-server
    ```

4.  设置 mysql 服务

    开启数据库

    ```bash
    systemctl start mysqld
    ```

    设置开机自启

    ```bash
    systemctl enable mysqld
    ```

5.  查看初始化密码（通过日志文件）

    ```bash
    vim /var/log/mysqld.log
    ```

    输入 `/password` 查看密码

6.  登录后修改密码：

    由于mysql5.7默认密码策略比较严格，可以通过一下两个指令来加简化密码设置

    ```bash
    # 设置默认策略为0
    set global validate_password_policy=0;
    # 设置密码长度至少大于1位
    set global validate_password_length=1;
    ```

    如果上述无法设置，则先设置一个长度为8位，带有大小写、数字与特殊字符的密码，再操作一遍。

    然后设置密码

    ```bash
    set password for root@localhost = password('root'); 
    ```

### 设置开放远程登陆

mysql中开放远程登陆，并刷新

```bash
# 开放远程登陆
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;

# 刷新授权
flush privileges;
```

Linux 开放端口

```bash
# 开放3306端口
firewall-cmd --add-port=3306/tcp --permanent

# 重载防火墙配置
firewall-cmd --reload
```

### 安装后目录结构

| 路径                                                       | 参数         | 解释                           | 备注                            |
| ---------------------------------------------------------- | ------------ | ------------------------------ | ------------------------------- |
| /usr/bin                                                   | --basedir    | 相关命令目录                   | mysqladmin<br/>mysqldump 等命令 |
| /var/lib/mysql/                                            | --datadir    | mysql 数据库文件的存放路径     |                                 |
| /usr/lib64/mysql/plugin                                    | --plugin-dir | mysql 插件存放路径             |                                 |
| /var/log/mysqld.log                                        | --log-error  | mysql 错误日志路径             |                                 |
| /var/run/mysqld/mysqld.pid                                 | --pid-file   | 进程 pid 文件                  |                                 |
| /var/lib/mysql/mysql.sock                                  | --socket     | 本地连接时用的 unix 套接字文件 |                                 |
| /usr/share/mysql                                           |              | 配置文件目录                   | mysql 脚本及配置文件            |
| /etc/systemd/system/multi-user.target.wants/mysqld.service |              | 服务启停相关脚本               |                                 |
| /etc/my.cnf                                                |              | mysql 配置文件                 |                                 |

### 常见问题

#### 中文乱码：

通过SHOW VARIABLES LIKE '%char%'查看数据库编码问题,会发现默认latin1字符编码，不支持中文。

结局方案：

1.  修改 my.cnf 文件并在 mysqld 字节最后添加编码设置，目录为 `vim/etc/my.cnf`

    ```bash
    #在[mysqld]节点最后加上中文字符集配置
    character_set_server=utf8
    ```

2.  重启 mysql 服务器

    ```bash
    systemctl restart mysqld
    ```

3.  修改已生成的库表字符集

    ```bash
    # 修改以前数据库的字符集
    alter database test character set 'utf8';
    # 修改以前数据表的字符集
    alter table book convert tocharacter set 'utf8';
    ```



## Redis 安装部署

### 安装步骤

1.  gcc 语言环境：

    ```bash
    yum install -y gcc
    ```

2.  解压 redis 的 gz 文件

    ```bash
    tar -zxvf redis-5.0.5.tar.gz -C <要解压的目录>
    ```

3.  进入解压好的文件夹，编译并安装redis文件

    ```bash
    make MALLOC=libc
    make install PREFIX=<要安装的目录>
    ```

4.  配置环境变量

    ```bash
    # 1. 编辑 环境变量
    vim /etc/profile
    
    # 2. 添加环境变量
    export REDIS_HOME=<redis的安装目录>
    export PATH=$PATH:$REDIS_HOME/bin
    
    # 3. 刷新profile文件
    source /etc/profile
    ```

5.  可以设置后台启动，更改其配置文件 `redis.conf`

    ```bash
    daemonize yes
    ```

    然后启动 `redis-server` 时以指定配置文件启动

    ```bash
    # 假设 redis.conf 在安装目录下
    /usr/local/redis/bin/redis-server /redis/local/redis/bin/redis.conf
    ```

### 使用 Redis

安装好redis后，发现redis的安装目录下多了一个src目录，该目录就是redis编译安装完成之后的目录，有关redis服务启动，客户端启动的文件都在此目录下。

#### 启动redis的服务端

进入src目录下，输入如下命令即可启动redis服务端

```markdown
./redis-server [配置文件目录]
```

-   配置文件目录可以指定（支持相对路径），默认使用redis-server中的shell脚本的配置

![image-20201018151004428](_images/image-20201018151004428-1605530412734.png)

#### 启动redis客户端

进入src目录下，输入如下命令即可启动redis客户端

```
./redis-cli [-h hostname] [-p port]
```

-   -h：指定主机名，缺省值为本机ip
-   -p：指定端口号，缺省值为6379

![image-20201018151015849](_images/image-20201018151015849-1605530412735.png)

**客户端显示中文**

```
./redis-cli  -p 7000 --raw
```

### 设置开放远程连接

开放连接：找到redis.conf 文件，更改如下配置；

```bash
# 绑定指定ip设为空
bind 0.0.0.0

# 关闭保护模式
protected-mode no
```

开放端口：防火墙开放redis的端口

```bash
firewall-cmd --zone=public --add-port=<redis端口号>/tcp --permanent
```

>   注意：需要重载防火墙配置 `firewall-cmd --reload`

### 设置开机启动

可以设置redis为开机启动，在 `/etc/rc.d/rc.local` 开机启动命令集中写入如下内容：

```bash
#让java环境变量在此执行文件执行之前生效。
source /etc/profile 

# 启动redis，指定配置文件，假设 redis.conf 在安装目录下
/usr/local/redis/bin/redis-server /usr/local/redis/redis.conf
```

然后给予该文件执行权限

```bash
chmod +x /etc/rc.d/rc.local
```



## Nginx 安装部署

