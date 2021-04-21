---
title: "Windowsserverenvironment"
date: 2021-04-21T18:24:40+08:00
lastmod: 2021-04-21T18:24:40+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

# windows env

## jdk

1. 安装目录  
    1. `D:\web_servers\yst_java\jdk`
    2. `D:\web_servers\yst_java\jre`
2. 设置环境变量  
    1. 添加新系统变量JAVA_HOME  
    `JAVA_HOME=D:\web_server\yst_java\jdk`
    2. 在系统变量Path中添加新的变量  
    `Path：%JAVA_HOME%\bin;`
    3. 添加CLASSPATH  
    `.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;`
3. 


## tomcat

* 解压后，双击 `start.bat`

### 安装tomcat服务

1. 添加新用户 tomcatuser
2. 赋予 tomcatuser 对 java 安装目录的执行权限
3. 


### 修改配置文件

#### server.xml

* shutdown 端口， 改成 `-1`， 或者修改 `shutdown="SHUTDOWN"` 为 `shutdown="复杂密码"`
* 注释 ajp 端口配置 `<!-- -->`
* 取消自动部署 `unpackWARs="false" autoDeploy="false"`
* 修改连接器版本，不添加会显示 “Apache-Coyote/1.1”

    <Connector port="8080" ...
        server="Apache" />
* 添加编码 `URIEncoding="UTF-8"`

#### content.xml

* 修改tomcat 使用的内存大小

```xml
/conf/context.xml中的Context中添加
<Resources
    cachingAllowed="true"
    cacheMaxSize="100000"
/>
```


#### web.xml  

* 防止404时列出所有资源  

```xml
<init-param>
    <param-name>listings</param-name>
    <param-value>false</param-value>  <!-- 防止404时列出目录下所有资源 -->
</init-param>
```
* 默认错误页面会显示堆栈信息，添加一下后会出现空白页

```xml
$ welcome-file-list标签后添加以下信息
<error-page>
    <exception-type>java.lang.Throwable</exception-type>
    <location>/error.jsp</location>
</error-page>
```

#### 去除版本信息

```
解压CATALINA_HOME/server/lib/catalina.jar 
修改org\apache\catalina\util\ServerInfo.properties 
jar xf catalina.jar 
jar uf catalina.jar org/apache/catalina/util/ServerInfo.properties
```


### tomcat 服务启动

#### 安装到服务

```
service.bat install ydsc_ydg8666  / service.bat remove ydsc_ydg8666
```

#### 命令行启动 

```
runas /user:tomcateruser cmd.exe` ,输入密码就好了，密码不能粘贴复制, 执行 .\start.bat
```

#### 修改项目的权限

## redis

1. 安装服务  
    ```go
    redis-server --service-install redis.windows.conf --loglevel verbose
    ```

2. 启动停止服务  
    ```
    redis-server --service-start ； redis-server --service-stop
    ```
3. redis 多实例  
```bat
redis-server --service-install –service-name redis6379-ydsc redis.windows-service.conf --loglevel verbose
$可选项 `--port 10003` , `--maxmemory 200m` （默认单位是字节）
```

4. 卸载redis服务
    1. `redis-server --service-uninstall --service-name redis6379-ydsc`

## mysql

1. 创建安装目录和数据目录:   

    /path/to/mysql 和 /path/to/mysql_data

2. 复制mysql安装目录到  

    /path/to/mysql

3. 删除data目录下的所有内容
4. 在bin下执行    `mysqld.exe install mysql_name`
5. 修改配置文件中 basedir、 datadir 、port 等信息；  
    > 注意路径中为反斜线
6. 修改注册表读取自定义配置文件
    > cmd -->regedit --> HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\MYSQL_NAME
7. bin下执行 `mysql --initialize` 初始化数据库，据说 新的密码在错误日志中
8. 重置密码
    1. 配置文件中添加 skip-grant-tables
    2. mysql 5.7.6以后 `ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';`
    3. 5.7.5以前 `SET PASSWORD FOR 'root'@'localhost' = PASSWORD('MyNewPass');`
