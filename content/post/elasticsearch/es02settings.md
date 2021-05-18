---
title: "Es02settings"
date: 2021-04-21T18:08:29+08:00
lastmod: 2021-04-21T18:08:29+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

# 集群设置

官方支持的操作系统和JVM 矩阵可在此处获得： 支持矩阵。Elasticsearch在列出的平台上进行了测试，但它也可能在其他平台上运行。

Elasticsearch是使用Java构建的，并且包含来自每个发行版中的JDK维护者（GPLv2 + CE）的捆绑版本的 OpenJDK。捆绑的JVM是推荐的JVM，位于jdkElasticsearch主目录的目录中。

要使用您自己的Java版本，请设置JAVA_HOME环境变量。如果必须使用与捆绑的JVM不同的Java版本，我们建议使用受支持的 LTS版本的Java。如果使用已知错误的Java版本，Elasticsearch将拒绝启动。使用自己的JVM时，可能会删除捆绑的JVM目录。


## 安装Elasticsearch

tar.gz 包, 可以在 任意版本的linux 和macos上运行

windows.zip 在windows系统上运行

deb包, 在ubuntu debian 及其它 debian系linux操作系统上 deepinos

rpm包, 适合安装在Red Hat，Centos，SLES，OpenSuSE和其他基于RPM的系统上。RPM可以从Elasticsearch网站或我们的RPM存储库下载。

msi包, 该msi软件包适合安装在至少安装了.NET 4.5框架的Windows 64位系统上，并且是在Windows上开始使用Elasticsearch的最简单选择。MSI可以从Elasticsearch网站下载。

docker, 图像可用于将Elasticsearch作为Docker容器运行。它们可以从Elastic Docker Registry下载。

配置管理工具

[puppet](https://github.com/elastic/puppet-elasticsearch)

[chef](https://github.com/elastic/cookbook-elasticsearch)

[ansible](https://github.com/elastic/ansible-elasticsearch)

### 在Linux或MacOS上从归档安装Elasticsearch

```shell
$ linux os version
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.0.0-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.0.0-linux-x86_64.tar.gz.sha512
shasum -a 512 -c elasticsearch-7.0.0-linux-x86_64.tar.gz.sha512 
tar -xzf elasticsearch-7.0.0-linux-x86_64.tar.gz
cd elasticsearch-7.0.0 /
```

此目录被称为 `$ES_HOME`

```shell
$ macos version
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.0.0-darwin-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.0.0-darwin-x86_64.tar.gz.sha512
shasum -a 512 -c elasticsearch-7.0.0-darwin-x86_64.tar.gz.sha512 
tar -xzf elasticsearch-7.0.0-darwin-x86_64.tar.gz
cd elasticsearch-7.0.0 / 
```

X-Pack将尝试在Elasticsearch中自动创建多个索引。默认情况下，Elasticsearch配置为允许自动创建索引，不需要其他步骤。但是，如果你有Elasticsearch禁用自动创建索引，你必须配置 action.auto_create_index的elasticsearch.yml，让X-包创建以下指标：

action.auto_create_index: .monitoring*,.watches,.triggered_watches,.watcher-history*,.ml*

如果您使用的是Logstash 或Beats，那么您的action.auto_create_index设置中很可能需要其他索引名称，具体值取决于您的本地配置。如果您不确定环境的正确值，可以考虑设置 *允许自动创建所有索引的值。

从命令行中启动

`./bin/elasticsearch`

默认情况下，Elasticsearch在前台运行，将其日志打印到标准输出（stdout），然后按下即可停止Ctrl-C。

检查Elasticsearch是否正在运行编辑

```es
GET /
```

```shell
curl http://localhost:9200/
```

哪个应该给你一个像这样的回复：

```json
{
  "name" : "Cp8oag6",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "AT69_T_DTp-1qgIJlatQqA",
  "version" : {
    "number" : "7.0.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "f27399d",
    "build_date" : "2016-03-30T09:51:41.449Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "1.2.3",
    "minimum_index_compatibility_version" : "1.2.3"
  },
  "tagline" : "You Know, for Search"
}
```

stdout可以使用 命令行上的-q或--quiet选项禁用日志打印。

作为守护进程编辑运行

要将Elasticsearch作为守护程序运行，请-d在命令行中指定，并使用以下-p选项将进程ID记录在文件中：

`./bin/elasticsearch -d -p pid`

可以在$ES_HOME/logs/目录中找到日志消息。

要关闭Elasticsearch，请终止pid文件中记录的进程ID ：

`pkill -F pid`

RPM和Debian 软件包中提供的启动脚本负责为您启动和停止Elasticsearch进程。

在命令行编辑中配置Elasticsearch

$ES_HOME/config/elasticsearch.yml 默认情况下，Elasticsearch从文件加载其配置。[配置Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)中介绍了此配置文件的格式 。

可以在命令行中指定可在配置文件中指定的任何设置，使用以下-E语法：

`./bin/elasticsearch -d -Ecluster.name=my_cluster -Enode.name=node_1`

通常，cluster.name应将任何群集范围的设置（如cluster.name）添加到elasticsearch.yml配置文件中，而node.name可以在命令行上指定任何特定于节点的设置。

档案(archive)的目录布局

归档分发完全是独立的。默认情况下，所有文件和目录都包含在$ES_HOME 解压缩存档时创建的目录中。

这非常方便，因为您不必创建任何目录来开始使用Elasticsearch，卸载Elasticsearch就像删除$ES_HOME目录一样简单。但是，建议更改config目录，数据目录和logs目录的默认位置，以便以后不删除重要数据。

| Type | Description | Default Location | Setting |
| ------ | ------ | ------ | ------ |
| home | Elasticsearch主目录或 $ES_HOME | 通过解压缩归档创建的目录 |   |
| bin | 二进制脚本，包括elasticsearch启动节点和elasticsearch-plugin安装插件 | $ES_HOME/bin |   |
| conf | 配置文件包括 elasticsearch.yml | $ES_HOME/config | [ES_PATH_CONF](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#config-files-location) |
| data | 节点上分配的每个索引/分片的数据文件的位置。可以容纳多个位置。 | $ES_HOME/data | path.data |
| logs | 日志文件位置。 | $ES_HOME/logs | path.logs |
| plugins | 插件文件位置。每个插件都将包含在一个子目录中。 | $ES_HOME/plugins |   |
| repo | 共享文件系统存储库位置。可以容纳多个位置。文件系统存储库可以放在此处指定的任何目录的任何子目录中。 | 未配置 | path.repo |
| script | 脚本文件的位置。 | $ES_HOME/scripts | path.scripts |


#### 后续步骤

您现在已经设置了测试Elasticsearch环境。在开始认真开发或使用Elasticsearch投入生产之前，您必须进行一些额外的设置：

* 了解[如何配置Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)。
* 配置[重要的Elasticsearch设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html)。
* 配置重要的[系统设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html)。

均在第二章 配置elastic

### .zip在Windows上安装Elasticsearch

[在windows上安装es](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-windows.html)

这个就算了吧

### 使用Debian软件包安装Elasticsearch

[在debian安装](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html)

这个也算了吧

### 使用RPM安装Elasticsearch

[在redhat系上安装](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html)

过

### 使用Windows MSI安装程序安装Elasticsearch

[gui界面的方式](https://www.elastic.co/guide/en/elasticsearch/reference/current/windows.html)

不过讲了一些如何配置成服务的方式, 用windows server 的话可以看一下

过

### 使用Docker安装Elasticsearch

Elasticsearch也可用作Docker镜像。图像使用centos7作为基本图像。

有关所有已发布的Docker镜像和标记的列表，请访问 www.docker.elastic.co。源文件在 Github中。

这些图像可以在Elastic许可下免费使用。它们包含开源和免费商业功能以及付费商业功能。 开始为期30天的试用，试用所有付费商业功能。有关弹性许可级别的信息，请参阅“ 订阅”页面。

#### 拉取镜像

获取Docker的Elasticsearch就像docker pull对Elastic Docker注册表发出命令一样简单。

`docker pull docker.elastic.co/elasticsearch/elasticsearch:7.0.0`

或者，您可以下载仅包含Apache 2.0许可下可用功能的其他Docker映像。要下载图像，请访问 www.docker.elastic.co。

#### 从命令行编辑运行Elasticsearch

##### 开发模式编辑

使用以下命令可以快速启动Elasticsearch以进行开发或测试：

`docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.0.0`

##### 生产环境

该vm.max_map_count内核设置需要至少设置为262144用于生产。根据您的平台：

###### Linux的

该vm.max_map_count设置应永久设置为/etc/sysctl.conf：

```shell
$ grep vm.max_map_count /etc/sysctl.conf
vm.max_map_count=262144
```

要在实时系统类型上应用该设置： sysctl -w vm.max_map_count=262144

###### 使用Docker for Mac的 macOS

vm.max_map_count必须在xhyve虚拟机中设置该设置：

$ screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty

只需按Enter键并像配置sysctlLinux一样配置设置：

`sysctl -w vm.max_map_count=262144`

###### Windows和macOS与Docker Toolbox

vm.max_map_count必须通过docker-machine设置该设置：

```shell
docker-machine ssh
sudo sysctl -w vm.max_map_count=262144
```

以下示例显示包含两个Elasticsearch节点的集群。要打开群集，请使用 docker-compose.yml并输入：

`docker-compose up`

docker-compose在Linux上没有预装, 在安装Docker时。可以在[Docker Compose网页](https://docs.docker.com/compose/install/#install-using-pip)上找到安装它的说明 。

节点es01在通过Docker网络进行通话localhost:9200时es02 进行侦听es01。

这个例子也使用 docker named volume ，并被命名esdata01和esdata02创建 如果尚未创建。

docker-compose.yml:

```yml
version: '2.2'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.0.0
    container_name: es01
    environment:
      - node.name=es01
      - discovery.seed_hosts=es02
      - cluster.initial_master_nodes=es01,es02
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - esnet
  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.0.0
    container_name: es02
    environment:
      - node.name=es02
      - discovery.seed_hosts=es01
      - cluster.initial_master_nodes=es01,es02
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata02:/usr/share/elasticsearch/data
    networks:
      - esnet

volumes:
  esdata01:
    driver: local
  esdata02:
    driver: local

networks:
  esnet:
```

要停止群集，请键入docker-compose down。数据卷将保持不变，因此可以使用相同的数据再次启动集群 docker-compose up。要销毁群集和数据卷，只需键入即可 docker-compose down -v。

#### 检查集群状态

```shell
curl http://127.0.0.1:9200/_cat/health
1472225929 15:38:49 docker-cluster green 2 2 4 2 0 0 0 0 - 100.0%
```

日志消息转到控制台，由配置的Docker日志记录驱动程序处理。默认情况下，您可以使用docker logs。

#### 使用Docker 编辑配置Elasticsearch

Elasticsearch从其下的文件加载其配置/usr/share/elasticsearch/config/。配置Elasticsearch和设置JVM选项中记录了这些配置文件。

该图像提供了几种配置Elasticsearch设置的方法，传统方法是提供自定义文件，也就是说 elasticsearch.yml，也可以使用环境变量来设置选项：

##### A.通过Docker环境变量编辑显示参数

例如，要定义群集名称，docker run您可以传递 -e "cluster.name=mynewclustername"。双引号是必需的。

##### B.绑定配置编辑

创建自定义配置文件并将其挂载到映像的相应文件上。例如结合安装一custom_elasticsearch.yml与docker run可与参数来完成：

`-v full_path_to/custom_elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml`

容器使用 `uid:gid 1000:1000` 以用户身份运行Elasticsearchelasticsearch。此用户需要可以访问绑定的挂载主机目录和文件（ custom_elasticsearch.yml如上所述）。对于数据和日志目录，例如，也需要写访问。另见下面的注释1。/usr/share/elasticsearch/data

##### C.自定义镜像

在某些环境中，准备包含配置的自定义镜像可能更有意义。使用一个 Dockerfile实现这一点可能很简单：

```dockerfile
FROM docker.elastic.co/elasticsearch/elasticsearch:7.0.0
COPY --chown=elasticsearch:elasticsearch elasticsearch.yml /usr/share/elasticsearch/config/
```

然后，您可以使用以下内容构建和尝试图像：

```shell
docker build --tag=elasticsearch-custom .
docker run -ti -v /usr/share/elasticsearch/data elasticsearch-custom
```

某些插件需要其他安全权限。您必须通过tty在运行Docker镜像时附加a 并在提示时接受yes，或者单独检查安全权限以及您是否愿意将--batch标志添加到plugin install命令来明确接受它们。有关 详细信息，请参阅[插件管理文档](https://www.elastic.co/guide/en/elasticsearch/plugins/7.0/_other_command_line_parameters.html)。

##### D.覆盖镜像默认的 CMD 命令

通过覆盖镜像的默认命令，可以将选项作为命令行选项传递给Elasticsearch进程。例如：

`docker run <various parameters> bin/elasticsearch -Ecluster.name=mynewclustername`

##### 使用Elasticsearch Docker映像编辑配置SSL / TLS

[请参阅加密Elasticsearch Docker容器中的通信](https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-tls-docker.html)

##### 生产使用注意事项和默认编辑

我们收集了许多生产用途的最佳实践。下面提到的任何Docker参数都假定使用docker run。

###### 1.默认情况下，Elasticsearch elasticsearch使用 `uid：gid 1000:1000` 作为用户在容器内运行。

> 警告
> 一个例外是Openshift，它使用任意分配的用户ID运行容器。Openshift将使用gid设置呈现持久性卷，0无需任何调整即可使用。

如果要绑定安装本地目录或文件，请确保此用户可以读取，而数据和日志目录还需要写访问权限。一个好的策略是授予组 `gid 1000或0` 访问本地目录的权限。例如，要准备一个本地目录来通过bind-mount存储数据：

```shell
mkdir esdatadir
chmod g+rwx esdatadir
chgrp 1000 esdatadir
```

作为最后的手段，您还可以强制容器通过环境变量来改变用于数据和日志目录的任何绑定装载的所有权TAKE_FILE_OWNERSHIP。在这种情况下，它们将由 `uid:gid 1000:0` 拥有 ，根据需要提供对Elasticsearch进程的读/写访问权限.

###### 2. 确保为elasticsearch容器提供nofile和nproc的增加的ulimits非常重要 。

验证 Docker守护程序的init系统是否已将这些设置为可接受的值，并在需要时在守护程序中调整它们，或者为每个容器覆盖它们，例如使用docker run：

`--ulimit nofile=65535:65535`

检查上述ulimits的Docker守护程序默认值的一种方法是运行：

`docker run --rm centos：7 / bin / bash -c'ulimit -Hn && ulimit -Sn && ulimit -Hu && ulimit -Su'`

###### 3. 需要禁用交换以提高性能和节点稳定性。

这可以通过[Elasticsearch文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html)中提到的任何方法来实现 。

如果你选择这种 `bootstrap.memory_lock: true` 方法，除了通过任何配置方法定义它之外，你还需要 `memlock: true` ulimit，无论是在Docker守护进程中定义的 还是专门为容器设置的。

这在上面的 `docker-compose.yml` 中进行了 演示。如果使用docker run：

`-e "bootstrap.memory_lock=true" --ulimit memlock=-1:-1`

###### 4. 该图像公开 TCP端口9200和9300.

对于群集，建议使用随机化已发布的端口--publish-all，除非您为每个主机固定一个容器。

###### 5. 使用ES_JAVA_OPTS环境变量设置堆大小。

例如，使用16GB，使用-e ES_JAVA_OPTS="-Xms16g -Xmx16g"与docker run。

###### 6. 例如，将部署固定到特定版本的Elasticsearch Docker映像

docker.elastic.co/elasticsearch/elasticsearch:7.0.0。

###### 7. 始终使用绑定的卷/usr/share/elasticsearch/data，如生产示例中所示， 原因如下：

* 如果容器被杀死，Elasticsearch节点的数据不会丢失
* Elasticsearch对I / O敏感，Docker存储驱动程序不适合快速I / O.
* 它允许使用高级 Docker卷插件

###### 8. 如果您使用的是devicemapper存储驱动程序，请确保未使用默认loop-lvm模式。

配置docker-engine 代替使用 direct-lvm。

###### 9. 请考虑使用其他日志记录驱动程序集中日志 。

另请注意，默认的json-file日志记录驱动程序不适合生产使用。

## 配置Elasticsearch

Elasticsearch具有良好的默认值，只需要很少的配置。可以使用Cluster Update Settings API 在正在运行的群集上更改大多数设置 。

配置文件应包含特定于节点的设置（例如node.name和路径），或节点为了能够加入群集而需要的设置，例如cluster.name和network.host。

### 配置文件位置

Elasticsearch有三个配置文件：

* elasticsearch.yml 用于配置Elasticsearch
* jvm.options 用于配置Elasticsearch JVM设置
* log4j2.properties 用于配置Elasticsearch日志记录

这些文件位于config目录中，其默认位置取决于安装是来自存档分发（tar.gz或 zip）还是包分发（Debian或RPM软件包）。

对于归档分发，config目录位置默认为 $ES_HOME/config。可以通过ES_PATH_CONF环境变量更改config目录的位置， 如下所示：

`ES_PATH_CONF=/path/to/my/config ./bin/elasticsearch`

或者，您可以通过命令行或通过shell配置文件来export获取ES_PATH_CONF环境变量。

对于包分发，config目录位置默认为 /etc/elasticsearch。config目录的位置也可以通过ES_PATH_CONF环境变量进行更改，但请注意，在shell中设置它是不够的。相反，此变量来自 /etc/default/elasticsearch（对于Debian软件包）和 /etc/sysconfig/elasticsearch（对于RPM软件包）。您需要相应地编辑ES_PATH_CONF=/etc/elasticsearch其中一个文件中的 条目以更改配置目录位置。


### 配置文件格式编辑

配置格式为YAML。以下是更改数据路径和日志目录的示例：

```yaml
path:
    data: /var/lib/elasticsearch
    logs: /var/log/elasticsearch
```

设置也可以按如下方式展平：

```yaml
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```

### 环境变量替换编辑

使用${...}配置文件中的符号引用的环境变量将替换为环境变量的值，例如：

```yaml
node.name:    ${HOSTNAME}
network.host: ${ES_NETWORK_HOST}
```

### 设置JVM选项

您应该很少需要更改Java虚拟机（JVM）选项。如果这样做，最可能的更改是设置堆大小。本文档的其余部分详细说明了如何设置JVM选项。

设置JVM选项（包括系统属性和JVM标志）的首选方法是通过jvm.options配置文件。此文件的默认位置是config/jvm.options（从tar或zip发行版/etc/elasticsearch/jvm.options安装时）和（从Debian或RPM软件包安装时）。

此文件包含遵循特殊语法的以行分隔的JVM参数列表：

* 仅包含空格的行被忽略
* 以...开头的行#被视为注释，并被忽略
  * `＃这是评论`
* 以一个 - 开头的行被视为JVM选项，该选项独立于JVM的版本而应用
  * `-Xmx2g`
* 以数字开头后跟一个 `:` 后跟 一个 `-` 的行-被视为JVM选项，仅当JVM的版本与数字匹配时才适用
  * `8:-Xmx2g`
* 以数字开头后跟一个 `-` 后跟一个 `:` 的行 被视为JVM选项，仅当JVM的版本大于或等于数字时才适用
  * `8-:-Xmx2g`
* 以数字开头后跟一个 `-` 后跟数字后跟一个 `:` 的行, 被视为JVM选项，仅当JVM的版本落在两个数字的范围内时才适用
  * `8-9:-Xmx2g`
* 所有其他行都被拒绝了

您可以将自定义JVM标志添加到此文件，并将此配置检查到版本控制系统中。

设置Java虚拟机选项的另一种机制是通过 ES_JAVA_OPTS环境变量。例如：

```shell
export ES_JAVA_OPTS="$ES_JAVA_OPTS -Djava.io.tmpdir=/path/to/temp/dir"
./bin/elasticsearch
```

使用RPM或Debian软件包时，ES_JAVA_OPTS可以在系统配置文件中指定 。

JVM具有用于观察JAVA_TOOL_OPTIONS 环境变量的内置机制。我们故意在包装脚本中忽略此环境变量。主要原因是在某些操作系统（例如，Ubuntu）上，默认情况下通过此环境变量安装了代理，我们不希望干扰Elasticsearch。

此外，一些其他Java程序支持JAVA_OPTS环境变量。这不是 JVM中内置的机制，而是生态系统中的约定。但是，我们不支持此环境变量，而是支持通过jvm.options文件或环境变量设置JVM选项ES_JAVA_OPTS，如上所述。

`/etc/sysconfig/elasticsearch`

或 `/etc/default/elasticsearch`

### [安全设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html)

某些设置是敏感的，依靠文件系统权限来保护其值是不够的。对于此用例，Elasticsearch提供了一个密钥库和elasticsearch-keystore管理密钥库中设置的工具。

此处的所有命令都应该作为运行Elasticsearch的用户运行。

只有一些设置可以从密钥库中读取。请参阅每个设置的文档，以查看它是否作为密钥库的一部分受支持。

只有在重新启动Elasticsearch之后，对密钥库的所有修改才会生效。

elasticsearch密钥库目前仅提供模糊处理。将来，将添加密码保护。

这些设置与elasticsearch.yml配置文件中的常规设置一样，需要在集群中的每个节点上指定。目前，所有安全设置都是特定于节点的设置，每个节点上的值必须相同。

#### 创建密钥库编辑

要创建elasticsearch.keystore，请使用以下create命令：

`bin/elasticsearch-keystore create`

该文件elasticsearch.keystore将与旁边一起创建elasticsearch.yml。

#### 列出密钥库编辑中的设置

使用以下list命令可以获得密钥库中的设置列表：

`bin/elasticsearch-keystore list`

#### 添加字符串设置编辑

可以使用以下add命令添加敏感字符串设置，例如云插件的身份验证凭据：

`bin/elasticsearch-keystore add the.setting.name.to.set`

该工具将提示设置的值。要通过stdin传递值，请使用--stdin标志：

`cat /file/containing/setting/value | bin/elasticsearch-keystore add --stdin the.setting.name.to.set`

#### 添加文件设置编辑

您可以使用该add-file命令添加敏感文件，例如云插件的身份验证密钥文件。确保在设置名称后包含文件路径作为参数。

`bin / elasticsearch-keystore add-file the.setting.name.to.set /path/example-file.json/ elasticsearch - keystore add - file the 。设置。名字。到。set / path / example - file 。JSON `

#### 删除设置编辑

要从密钥库中删除设置，请使用以下remove命令：

`bin/elasticsearch-keystore add-file the.setting.name.to.set /path/example-file.json`

#### 可重新加载的安全设置编辑

与设置值一样elasticsearch.yml，密钥库内容的更改不会自动应用于正在运行的elasticsearch节点。重新读取设置需要重新启动节点。但是，某些安全设置被标记为可重新加载。可以重新读取这些设置并将其应用于正在运行的节点上。

所有安全设置的值（可重新加载或不可加载）在所有群集节点上必须相同。在进行所需的安全设置更改后，使用该bin/elasticsearch-keystore add命令调用：

`POST _nodes/reload_secure_settings`

此API将在每个群集节点上解密并重新读取整个密钥库，但仅应用可重新加载的安全设置。对其他设置的更改将在下次重新启动后生效。一旦调用返回，重载就已完成，这意味着所有依赖于这些设置的内部数据结构都已更改。一切看起来好像设置从一开始就具有新值。

更改多个可重新加载的安全设置时，请在每个群集节点上修改所有这些设置，然后发出reload_secure_settings调用，而不是在每次修改后重新加载。

### 日志配置

Elasticsearch使用Log4j 2进行日志记录。可以使用log4j2.properties文件配置Log4j 2。Elasticsearch公开三个属性${sys:es.logs.base_path}， ${sys:es.logs.cluster_name}以及${sys:es.logs.node_name}可以在配置文件中被引用，以确定日志文件的位置。该属性${sys:es.logs.base_path}将解析为日志目录， ${sys:es.logs.cluster_name}将解析为群集名称（在默认配置中用作日志文件名的前缀）， ${sys:es.logs.node_name}并将解析为节点名称（如果明确设置了节点名称）。

例如，如果你的日志目录（path.logs）是/var/log/elasticsearch和您的群集名为production然后${sys:es.logs.base_path}将解析/var/log/elasticsearch和 ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}.log 将解析/var/log/elasticsearch/production.log。

```log
######## Server JSON ############################
appender.rolling.type = RollingFile  # 配置滚动日志
appender.rolling.name = rolling
appender.rolling.fileName = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}_server.json # 日志输出位置
appender.rolling.layout.type = ESJsonLayout # 日志格式
appender.rolling.layout.type_name = server # type_name是一个填充该type字段的标志ESJsonLayout。在解析它们时，它可以更容易地区分不同类型的日志。
appender.rolling.filePattern = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}-%d{yyyy-MM-dd}-%i.json.gz  # 滚动日志到/var/log/elasticsearch/production-yyyy-MM-dd-i.json; 日志将在每个卷上压缩i并将递增
appender.rolling.policies.type = Policies
appender.rolling.policies.time.type = TimeBasedTriggeringPolicy  # 使用基于时间的滚动策略
appender.rolling.policies.time.interval = 1 # 每天滚动日志
appender.rolling.policies.time.modulate = true # 在日界上对齐卷（而不是每隔二十四小时滚动）
appender.rolling.policies.size.type = SizeBasedTriggeringPolicy # 使用基于大小的滚动策略
appender.rolling.policies.size.size = 256MB # 256 MB后滚动日志
appender.rolling.strategy.type = DefaultRolloverStrategy 
appender.rolling.strategy.fileIndex = nomax
appender.rolling.strategy.action.type = Delete # 滚动日志时使用删除操作
appender.rolling.strategy.action.basepath = ${sys:es.logs.base_path}
appender.rolling.strategy.action.condition.type = IfFileName # 仅删除与文件模式匹配的日志
appender.rolling.strategy.action.condition.glob = ${sys:es.logs.cluster_name}-* # 该模式仅删除主日志
appender.rolling.strategy.action.condition.nested_condition.type = IfAccumulatedFileSize # 仅在我们累积了太多压缩日志时才删除
appender.rolling.strategy.action.condition.nested_condition.exceeds = 2GB # 压缩日志的大小条件为2 GB
################################################
```

```text
######## Server -  old style pattern ###########
appender.rolling_old.type = RollingFile
appender.rolling_old.name = rolling_old
appender.rolling_old.fileName = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}_server.log 
appender.rolling_old.layout.type = PatternLayout
appender.rolling_old.layout.pattern = [%d{ISO8601}][%-5p][%-25c{1.}] [%node_name]%marker %m%n
appender.rolling_old.filePattern = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}-%d{yyyy-MM-dd}-%i.old_log.gz
```

old style模式追加器 的配置。这些日志将保存在*.log文件中，如果存档将保存在文件中* .log.gz。请注意，这些应被视为已弃用，将来会被删除。

Log4j的配置解析被任何无关的空白混淆了; 如果您在此页面上复制并粘贴任何Log4j设置，或者一般输入任何Log4j配置，请务必修剪任何前导和尾随空格。

注意，除了可以更换.gz用.zip在appender.rolling.filePattern压缩使用zip格式轧制日志。如果删除.gz 扩展名，则日志不会在滚动时进行压缩。

如果要在指定的时间段内保留日志文件，可以使用具有删除操作的翻转策略。

```text
appender.rolling.strategy.type = DefaultRolloverStrategy # 	配置 DefaultRolloverStrategy
appender.rolling.strategy.action.type = Delete # 配置Delete处理翻转的操作
appender.rolling.strategy.action.basepath = ${sys:es.logs.base_path} # Elasticsearch日志的基本路径
appender.rolling.strategy.action.condition.type = IfFileName # 处理翻转时应用的条件
appender.rolling.strategy.action.condition.glob = ${sys:es.logs.cluster_name}-*  #从与glob匹配的基本路径中删除文件 ${sys:es.logs.cluster_name}-*; 这是日志文件滚动到的glob; 这只需要删除已滚动的Elasticsearch日志，但也不需要删除弃用和慢速日志
appender.rolling.strategy.action.condition.nested_condition.type = IfLastModified # 应用于与glob匹配的文件的嵌套条件
appender.rolling.strategy.action.condition.nested_condition.age = 7D # 保留日志七天
```

可以加载多个配置文件（在这种情况下，它们将被合并），只要它们被命名log4j2.properties并将Elasticsearch配置目录作为祖先; 这对于暴露其他记录器的插件很有用。记录器部分包含java包及其相应的日志级别。appender部分包含日志的目标。有关如何自定义日志记录和所有支持的appender的详细信息，请参阅 Log4j文档。

#### 配置日志级别

有四种配置日志记录级别的方法，每种方法都有适合使用的情况。

1. 通过命令行:  
    ( -E <name of logging hierarchy>=<level>例如， -E logger.org.elasticsearch.transport=trace）。当您临时调试单个节点上的问题时（例如，启动问题或开发期间），这是最合适的。
1. 通过elasticsearch.yml:  
    ( <name of logging hierarchy>: <level>例如， logger.org.elasticsearch.transport: trace）。当您临时调试问题但未通过命令行启动Elasticsearch（例如，通过服务）或者您希望更长期地调整日志记录级别时，这是最合适的。
1. 通过群集设置：  
  ```yml
  PUT /_cluster/settings
  {
    "transient": {
      "<name of logging hierarchy>": "<level>"
    }
  }
  ```
  例如：
  ```yml
  PUT /_cluster/settings
  {
    "transient": {
      "logger.org.elasticsearch.transport": "trace"
    }
  }
  ```
  当您需要动态调整正在运行的群集上的日志记录级别时，这是最合适的。
1. 通过log4j2.properties：
  ```shell
  logger.<unique_identifier>.name = <name of logging hierarchy>
  logger.<unique_identifier>.level = <level>
  ```
  例如:
  ```shell
  logger.transport.name = org.elasticsearch.transport
  logger.transport.level = trace
  ```
  当您需要对记录器进行细粒度控制时（例如，您希望将记录器发送到另一个文件，或以不同方式管理记录器;这是一种罕见的用例），这是最合适的。

#### 弃用日志记录

除常规日志记录外，Elasticsearch还允许您启用已弃用操作的日志记录。例如，如果您将来需要迁移某些功能，这可以让您尽早确定。默认情况下，将在WARN级别启用弃用日志记录，该级别是将发出所有弃用日志消息的级别。

`logger.deprecation.level = warn`

这将在日志目录中创建每日滚动弃用日志文件。定期检查此文件，尤其是当您打算升级到新的主要版本时。

默认日志记录配置已将弃用日志的卷策略设置为在1 GB后滚动和压缩，并最多保留五个日志文件（四个滚动日志和活动日志）。

您可以config/log4j2.properties通过将弃用日志级别设置为，在文件中禁用它error。

#### JSON日志格式

为了更轻松地解析Elasticsearch日志，现在以JSON格式打印日志。这是由Log4J布局属性配置的appender.rolling.layout.type = ESJsonLayout。此布局需要type_name设置一个属性，用于在解析时区分日志流。

```yml
appender.rolling.layout.type = ESJsonLayout
appender.rolling.layout.type_name = server
```

每行包含一个配置了属性的JSON文档ESJsonLayout。有关更多详细信息，请参阅此类javadoc。但是，如果JSON文档包含异常，则它将在多行上打印。第一行将包含常规属性，后续行将包含格式化为JSON数组的stacktrace。

您仍然可以使用自己的自定义布局。为此，请appender.rolling.layout.type使用不同的布局替换该行 。见下面的示例：

```yml
appender.rolling.type = RollingFile
appender.rolling.name = rolling
appender.rolling.fileName = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}_server.log
appender.rolling.layout.type = PatternLayout
appender.rolling.layout.pattern = [%d{ISO8601}][%-5p][%-25c{1.}] [%node_name]%marker %.-10000m%n
appender.rolling.filePattern = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}-%d{yyyy-MM-dd}-%i.log.gz
```

### 审核设置

xpack 付费 咱还是算了, 找个别的

#### 审核安全设置

所有这些设置都可以添加到elasticsearch.yml配置文件中。有关更多信息，请参阅 审核安全事件。

#### 一般审核设置

xpack.security.audit.enabled

设置为true在节点上启用审核。默认值为false。这会将审计事件<clustername>_audit.json 放在每个节点上命名的专用文件中。有关更多信息，请参阅配置日志级别编辑。

#### 审核的事件设置编辑

可以使用以下设置控制事件和有关记录内容的其他一些信息：

xpack.security.audit.logfile.events.include

指定要包含在审计输出中的事件。默认值为： access_denied, access_granted, anonymous_access_denied, authentication_failed, connection_denied, tampered_request, run_as_denied, run_as_granted。

xpack.security.audit.logfile.events.exclude

从输出中排除指定的事件。默认情况下，不排除任何事件。

xpack.security.audit.logfile.events.emit_request_body

指定是否在某些事件类型（例如）上包含来自REST请求的请求正文authentication_failed。默认值为false。

#### 重要

审计时不执行过滤，因此在审计事件中包含请求主体时，可以以纯文本方式审计敏感数据。

#### 本地节点信息设置编辑

xpack.security.audit.logfile.emit_node_name

指定是否将节点名称包含在每个审核事件中作为字段。默认值为true。

xpack.security.audit.logfile.emit_node_host_address

指定是否将节点的IP地址作为每个审核事件中的字段包含在内。默认值为false。

xpack.security.audit.logfile.emit_node_host_name

指定是否将节点的主机名作为字段包含在每个审核事件中。默认值为false。

xpack.security.audit.logfile.emit_node_id

指定是否将节点标识包含在每个审核事件中作为字段。这仅适用于新格式。也就是说，该信息不存在于<clustername>_access.log文件中。与节点名称不同，如果管理员更改配置文件中的设置，其值可能会更改，则节点ID将在群集重新启动时保持不变，管理员无法更改它。默认值为true。

#### 审核日志文件事件忽略策略编辑

这些设置会影响忽略策略 ，这些策略可以对打印到日志文件的审核事件进行细粒度控制。具有相同策略名称的所有设置组合在一起形成单个策略。如果事件与特定策略的所有条件匹配，则会忽略该事件并且不会打印。

xpack.security.audit.logfile.events.ignore_filters.<policy_name>.users

用户名或通配符列表。指定的策略不会为匹配这些值的用户打印审核事件。

xpack.security.audit.logfile.events.ignore_filters.<policy_name>.realms

身份验证领域名称或通配符的列表。指定的策略不会为这些领域中的用户打印审核事件。

xpack.security.audit.logfile.events.ignore_filters.<policy_name>.roles

角色名称或通配符列表。指定的策略不会为具有这些角色的用户打印审核事件。如果用户具有多个角色，其中一些角色不在策略范围内，则该策略 不会涵盖此事件。

xpack.security.audit.logfile.events.ignore_filters.<policy_name>.indices

索引名称或通配符列表。当事件中的所有索引都匹配这些值时，指定的策略不会打印审核事件。如果该事件涉及多个指数，其中一些指标 不在政策范围内，则该政策不会涵盖此事件。

### 跨群集复制设置 xpack

可以使用群集更新设置API在实时群集上动态更新这些跨群集复制设置 。

#### 远程恢复设置

以下设置可用于对远程恢复期间传输的数据进行速率限制 ：

ccr.indices.recovery.max_bytes_per_sec（动态）

限制每个节点上的总入站和出站远程恢复流量。由于此限制适用于每个节点，但可能有许多节点同时执行远程恢复，因此远程恢复字节的总量可能远高于此限制。如果将此限制设置得过高，则存在持续远程恢复将消耗可能会破坏群集稳定性的过多带宽（或其他资源）的风险。领导者和跟随者群集都使用此设置。例如，如果将其设置为20mb领导者，则领导者将仅发送20mb/s给关注者，即使关注者正在请求并且可以接受60mb/s。默认为40mb。

#### 高级远程恢复设置

可以设置以下专家设置来管理远程恢复所消耗的资源：

ccr.indices.recovery.max_concurrent_file_chunks（动态）

  控制每次恢复可并行发送的文件块请求数。由于多个远程恢复可能已经并行运行，因此增加此专家级设置可能仅在单个分片的远程恢复未达到配置的总入站和出站远程恢复流量的情况下有用

  ccr.indices.recovery.max_bytes_per_sec。默认为5。允许的最大值是10。

ccr.indices.recovery.chunk_size（动态）

  控制文件传输期间跟随者请求的块大小。默认为 1mb。

ccr.indices.recovery.recovery_activity_timeout（动态）

  控制恢复活动的超时。此超时主要适用于领导者群集。领导者集群必须在内存中打开资源，以便在恢复过程中向关注者提供数据。如果领导者在这段时间内没有收到关注者的恢复请求，它将关闭资源。默认为60秒。

ccr.indices.recovery.internal_action_timeout（动态）

  控制远程恢复过程中各个网络请求的超时。单个动作超时可能无法恢复。默认为60秒。

### 索引生命周期管理设置 xpack

这些索引级ILM设置通常通过索引模板进行配置。有关更多信息，请参阅设置策略编辑。

`index.lifecycle.name`

  用于管理索引的策略的名称。

`index.lifecycle.rollover_alias`

  索引翻转时要更新的索引别名。指定何时使用包含翻转操作的策略。当索引翻转时，别名会更新以反映索引不再是写入索引。有关翻转的详细信息，请参阅使用策略管理索引翻转。

### 许可证设置 xpack

您可以在elasticsearch.yml文件中配置此许可设置。有关更多信息，请参阅 许可证管理。

* xpack.license.self_generated.type
  * 设置为basic（默认）以启用基本X-Pack功能。
  * 如果设置为trial，则自生成许可证仅允许访问x-pack的所有功能30天。如果需要，您可以稍后将群集降级为基本许可证。

### 机器学习设置 xpack

您无需配置任何设置即可使用机器学习。它默认启用。

所有这些设置都可以添加到elasticsearch.yml配置文件中。还可以使用群集更新设置API在 群集中更新动态设置。

> 动态设置优先于elasticsearch.yml 文件中的设置。

#### 一般机器学习设置编辑

* `node.ml`
  * 设置为true（默认）以将节点标识为机器学习节点。
  * 如果设置为falsein elasticsearch.yml，则节点无法运行作业。如果设置为 true但xpack.ml.enabled设置为false，node.ml则忽略该设置，并且节点无法运行作业。如果要运行作业，则群集中必须至少有一个计算机学习节点。
  > 在专用协调节点或专用主节点上，禁用该node.ml角色。

* `xpack.ml.enabled`
  * 设置为true（默认）以在节点上启用机器学习。
  * 如果设置为falsein elasticsearch.yml，则在节点上禁用机器学习API。因此，节点无法打开作业，启动数据馈送或接收与机器学习API相关的传输（内部）通信请求。它还会影响连接到此Elasticsearch实例的所有Kibana实例; 您不需要在这些kibana.yml文件中禁用机器学习 。有关在特定Kibana实例中禁用机器学习的详细信息，请参阅 Kibana机器学习设置。
  > 如果要在群集中使用计算机学习功能，则必须 xpack.ml.enabled设置为true所有符合主节点的节点。这是默认行为。
* `xpack.ml.max_machine_memory_percent`
  
  * 机器学习可用于运行分析过程的机器内存的最大百分比。（这些进程与Elasticsearch JVM分开。）默认为 30百分比。限制基于机器的总内存，而不是当前的可用内存。如果这样做会导致机器学习作业的估计内存使用超过限制，则不会将作业分配给节点。
* `xpack.ml.max_model_memory_limit`
  
  * `model_memory_limit` 可以为此节点上的任何作业设置 的最大属性值。如果尝试创建model_memory_limit属性值大于此设置值的作业，则会发生错误。更新此设置时，现有作业不受影响。有关该model_memory_limit属性的更多信息 ，请参阅分析限制编辑。
* `xpack.ml.max_open_jobs`
  
  * 可以在节点上运行的最大作业数。默认为20。最大作业数也受内存使用量的限制，因此如果作业的估计内存使用量高于允许值，则在该节点上运行的作业数将少于此设置指定的数。
* `xpack.ml.node_concurrent_job_allocations`
  
  * opening每个节点上 可以同时处于该状态的最大作业数。通常情况下，作业在进入州之前会在此状态下花费少量时间open。必须在开放时恢复大型模型的工作在该opening州花费更多时间。默认为2。

#### 高级机器学习设置编辑

这些设置适用于高级用例; 默认值通常是足够的：

* xpack.ml.enable_config_migration（动态）
  
  * 保留。
* xpack.ml.max_anomaly_records（动态）
  
  * 每个桶输出的最大记录数。默认值为 500。
* xpack.ml.max_lazy_ml_nodes（动态）
  * 懒惰旋转机器学习节点的数量。在第一个机器学习作业打开之前不需要ML节点的情况下很有用。它默认为0并具有最大可接受值3。如果ML节点的当前数量>=大于此设置，则假设不再有可用的延迟节点，因为已经提供了所需数量的节点。如果在此设置设置为打开作业>0且没有可接受作业的节点，则作业将保持该OPENING状态，直到将新ML节点添加到群集并分配作业以在该节点上运行。
  > 此设置假定某些外部进程能够将ML节点添加到群集。此设置仅在与此类外部过程结合使用时才有用。

### 监控设置 xpack

默认情况下，启用监视但禁用数据收集。要启用数据收集，请使用该xpack.monitoring.collection.enabled设置。

您可以在elasticsearch.yml文件中配置这些监视设置。您还可以使用群集更新设置API动态设置其中一些 设置。

> 群集设置优先于elasticsearch.yml 文件中的设置。

要调整的监测数据如何显示在监控界面，配置 xpack.monitoring设置中 kibana.yml。要控制如何监控数据从收集Logstash，配置 xpack.monitoring设置 在logstash.yml。

有关更多信息，请参阅 [监视弹性堆栈](https://www.elastic.co/guide/en/elastic-stack-overview/7.0/xpack-monitoring.html)。

#### 常规监控设置

* xpack.monitoring.enabled
  * 设置为true（默认）以在节点上为Elasticsearch启用Elasticsearch X-Pack监视。
  > 要启用数据收集，还必须设置xpack.monitoring.collection.enabled 为true。它的默认值是false。

#### 监控集合设置

这些xpack.monitoring.collection设置控制如何从Elasticsearch节点收集数据。您可以使用群集更新设置API动态更改所有监视收集设置。

* xpack.monitoring.collection.enabled（动态）
  
  * [ 6.3.0 ] 在6.3.0中添加。 设置为true启用监视数据的收集。如果此设置为false（默认），则不会收集Elasticsearch监控数据，并忽略来自其他来源（如Kibana，Beats和Logstash）的所有监控数据。
* xpack.monitoring.collection.interval（动态）
  * -1从7.0.0开始，不再支持 设置为禁用数据收集。 [ 6.3.0 ] 在6.3.0中弃用。请改用xpack.monitoring.collection.enabledset false。
  * 控制收集数据样本的频率。默认为10s。如果修改收集间隔，请将xpack.monitoring.min_interval_seconds 选项设置kibana.yml为相同的值。
* xpack.monitoring.elasticsearch.collection.enabled（动态）
  
  * 控制是否应收集有关Elasticsearch集群的统计信息。默认为true。这与xpack.monitoring.collection.enabled不同，后者允许您启用或禁用所有监视集合。但是，此设置仅禁用Elasticsearch数据的收集，同时仍允许其他数据（例如，Kibana，Logstash，Beats或APM Server监视数据）通过此群集。
* xpack.monitoring.collection.cluster.stats.timeout（动态）
  
  * （时间值）收集群集统计信息的超时。默认为10s。
* xpack.monitoring.collection.node.stats.timeout（动态）
  
  * （时间值）收集节点统计信息的超时。默认为10s。
* xpack.monitoring.collection.indices（动态）
  
  * 控制监控从哪些索引收集数据。默认为所有索引。例如，将索引名称指定为逗号分隔列表test1,test2,test3。例如，名称可以包括通配符test*。您可以通过预先明确地明确排除索引-。例如，test*,-test3将监视test除以外的所有索引test3。像.security *或.kibana *这样的系统索引总是以a开头.，通常应该受到监控。考虑添加.*到索引列表中，确保监视系统索引。例如.*,test*,-test3
* xpack.monitoring.collection.index.stats.timeout（动态）
  
  * （ 时间值）收集索引统计信息的超时。默认为10s。
* xpack.monitoring.collection.index.recovery.active_only（动态）
  
  * 控制是否收集所有回收。设置为true仅收集活动恢复。默认为false。
* xpack.monitoring.collection.index.recovery.timeout（动态）
  
  * （时间值）收集恢复信息的超时。默认为10s。
* xpack.monitoring.history.duration（动态）
  * （时间值）保留持续时间，超过该持续时间，监视导出器创建的索引将自动删除。默认为7d（7天）。
  * 此设置的最小值为1d（1天），以确保正在监视某些内容，并且无法禁用该设置。
  > 此设置目前仅影响local类型导出器。使用http导出器创建的索引不会自动删除。
* xpack.monitoring.exporters
  
  * 配置代理存储监控数据的位置。默认情况下，代理使用本地导出程序来索引安装它的群集上的监视数据。使用HTTP导出器将数据发送到单独的监视集群。有关详细信息，请参阅本地导出程序设置， HTTP导出程序设置和 监视工作原理。

#### 本地导出器设置

该local出口是通过监控使用的默认出口国。正如名称所暗示的那样，它将数据导出到本地集群，这意味着没有太多需要配置。

如果您不提供任何导出器，则Monitoring将自动为您创建一个。如果提供了任何导出器，则不会添加默认值。

```yml
xpack.monitoring.exporters.my_local：my_local：
  类型：本地  类型：本地
```

* type
  * 本地导出器的值必须始终为local且是必需的。
* use_ingest
  * 是否向集群提供占位符管道以及每个批量请求的管道处理器。默认值为true。如果禁用，则表示它不会使用管道，这意味着将来的版本无法自动将批量请求升级到面向未来的版本。
* cluster_alerts.management.enabled
  * 是否为此群集创建群集警报。默认值为true。要使用此功能，必须启用Watcher。如果您具有基本许可证，则不会显示群集警报。

#### HTTP导出器设置

以下列出了可以与http导出器一起提供的设置。所有设置都显示为您为导出程序选择的名称后面的内容：

```yml
xpack.monitoring.exporters.my_remote：my_remote：
  类型：http  类型：http
  主持人：[“host：port”，...]主持人：[ “host：port” ，...]
```

* type
  * HTTP导出器的值必须始终http是必需的。
* host
  * 主机支持多种格式，包括阵列或单个值。支持的格式包括 hostname，hostname:port，http://hostname http://hostname:port，https://hostname，和 https://hostname:port。不能假设主机。默认方案始终http为默认端口，9200如果未作为host字符串的一部分提供，则始终为默认端口。

```yml
xpack.monitoring.exporters:
  example1:
    type: http
    host: "10.1.2.3"
  example2:
    type: http
    host: ["http://10.1.2.4"]
  example3:
    type: http
    host: ["10.1.2.5", "10.1.2.6"]
  example4:
    type: http
    host: ["https://10.1.2.3:9200"]
```

* auth.username
  
  * 如果auth.password提供了a，则需要用户名。
* auth.password
  
  * 的密码auth.username。
* connection.timeout
  
  * （时间值）HTTP连接应等待套接字为请求打开的时间量。默认值为6s。
* connection.read_timeout
  
  * （时间值）HTTP连接应等待套接字发回响应的时间量。默认值为10 * connection.timeout（60s如果两者都未设置）。
* ssl
  
  * 每个HTTP导出器都可以定义自己的TLS / SSL设置或继承它们。请参阅下面的 TLS / SSL部分。
* proxy.base_path
  
  * 为任何传出请求添加前缀的基本路径，例如/base/path（例如，批量请求将作为/base/path/_bulk）发送。没有默认值。
* headers
  * 添加到每个请求的可选标头，可以帮助通过代理路由请求。
    ```yml
    xpack.monitoring.exporters.my_remote:
      headers:
        X-My-Array: [abc, def, xyz]
        X-My-Header: abc123
    ```
  * 基于数组的标头的发送n时间n是数组的大小。Content-Type 并且Content-Length无法设置。Monitoring Agent创建的任何标头都将覆盖此处定义的任何内容。
* index.name.time_format
  
  * 默认情况下，每日监控索引更改默认日期后缀的机制。默认值为YYYY.MM.DD，这就是每日创建索引的原因。
* use_ingest
  
  * 是否向监视集群提供占位符管道，并为每个批量请求提供管道处理器。默认值为true。如果禁用，则表示它不会使用管道，这意味着将来的版本无法自动将批量请求升级到面向未来的版本。
* cluster_alerts.management.enabled
  
  * 是否为此群集创建群集警报。默认值为true。要使用此功能，必须启用Watcher。如果您具有基本许可证，则不会显示群集警报。
* cluster_alerts.management.blacklist
  * 阻止创建特定群集警报。它还会删除当前群集中已存在的所有适用手表。
  * 您可以将以下任何监视标识符添加到黑名单中：
    * elasticsearch_cluster_status
    * elasticsearch_version_mismatch
    * elasticsearch_nodes
    * kibana_version_mismatch
    * logstash_version_mismatch
    * xpack_license_expiration
  * 例如：["elasticsearch_version_mismatch","xpack_license_expiration"]。

#### X-Pack监控TLS / SSL设置

您可以配置以下TLS / SSL设置。如果未配置设置， 则使用默认TLS / SSL设置。

* xpack.monitoring.exporters.$NAME.ssl.supported_protocols
  * 支持的版本协议。有效协议：SSLv2Hello， SSLv3，TLSv1，TLSv1.1，TLSv1.2，TLSv1.3。TLSv1.3,TLSv1.2,TLSv1.1如果JVM支持TLSv1.3，则默认为，否则为TLSv1.2,TLSv1.1。
* xpack.monitoring.exporters.$NAME.ssl.verification_mode
  * 控制证书的验证。有效值是none， certificate和full。默认为full。
* xpack.monitoring.exporters.$NAME.ssl.cipher_suites
  * 支持的密码套件可以在Oracle的 Java Cryptography Architecture文档中找到。默认为``。

#### X-Pack监控TLS / SSL密钥和可信证书设置

以下设置用于指定在通过SSL / TLS连接进行通信时应使用的私钥，证书和可信证书。私钥和证书是可选的，如果服务器需要客户端身份验证进行PKI身份验证，则会使用私钥和证书。如果未指定以下任何设置，则使用默认TLS / SSL设置。

##### PEM编码文件

使用PEM编码文件时，请使用以下设置：

* xpack.monitoring.exporters.$NAME.ssl.key
  * 包含私钥的PEM编码文件的路径。
* xpack.monitoring.exporters.$NAME.ssl.key_passphrase
  * 用于解密私钥的密码。此值是可选的，因为密钥可能未加密。
* xpack.monitoring.exporters.$NAME.ssl.secure_key_passphrase（安全）
  * 用于解密私钥的密码。此值是可选的，因为密钥可能未加密。
* xpack.monitoring.exporters.$NAME.ssl.certificate
  * PEM编码文件的路径，其中包含将在请求时显示的证书（或证书链）。
* xpack.monitoring.exporters.$NAME.ssl.certificate_authorities
  * 应受信任的PEM编码证书文件的路径列表。

##### Java密钥库文件

使用包含应信任的私钥，证书和证书的Java密钥库文件（JKS）时，请使用以下设置：

* xpack.monitoring.exporters.$NAME.ssl.keystore.path
  * 保存私钥和证书的密钥库的路径。
* xpack.monitoring.exporters.$NAME.ssl.keystore.password
  * 密钥库的密码。
* xpack.monitoring.exporters.$NAME.ssl.keystore.secure_password（安全）
  * 密钥库的密码。
* xpack.monitoring.exporters.$NAME.ssl.keystore.key_password
  * 密钥库中私钥的密码。默认值为xpack.monitoring.exporters.$NAME.ssl.keystore.password。
* xpack.monitoring.exporters.$NAME.ssl.keystore.secure_key_password（安全）
  * 密钥库中私钥的密码。
* xpack.monitoring.exporters.$NAME.ssl.truststore.path
  * 信任库文件的路径。
* xpack.monitoring.exporters.$NAME.ssl.truststore.password
  * 信任库的密码。
* xpack.monitoring.exporters.$NAME.ssl.truststore.secure_password（安全）
  * 信任库的密码。

##### PKCS＃12文件

可以将Elasticsearch配置为使用包含应受信任的私钥，证书和证书的PKCS＃12容器文件（.p12或.pfx文件）。

PKCS＃12文件的配置方式与Java Keystore Files相同：

* xpack.monitoring.exporters.$NAME.ssl.keystore.path
  * 保存私钥和证书的PKCS＃12文件的路径。
* xpack.monitoring.exporters.$NAME.ssl.keystore.type
  * 将其设置PKCS12为表示密钥库是PKCS＃12文件。
* xpack.monitoring.exporters.$NAME.ssl.keystore.password
  * PKCS＃12文件的密码。
* xpack.monitoring.exporters.$NAME.ssl.keystore.secure_password（安全）
  * PKCS＃12文件的密码。
* xpack.monitoring.exporters.$NAME.ssl.keystore.key_password
  * 存储在PKCS＃12文件中的私钥的密码。默认值为xpack.monitoring.exporters.$NAME.ssl.keystore.password。
* xpack.monitoring.exporters.$NAME.ssl.keystore.secure_key_password（安全）
  * 存储在PKCS＃12文件中的私钥的密码。
* xpack.monitoring.exporters.$NAME.ssl.truststore.path
  * PKCS＃12文件的路径，用于保存要信任的证书。
* xpack.monitoring.exporters.$NAME.ssl.truststore.type
  * 将其设置PKCS12为表示信任库是PKCS＃12文件。
* xpack.monitoring.exporters.$NAME.ssl.truststore.password
  * PKCS＃12文件的密码。
* xpack.monitoring.exporters.$NAME.ssl.truststore.secure_password（安全）
  * PKCS＃12文件的密码。

#### PKCS＃11令牌

可以将Elasticsearch配置为使用包含应受信任的私钥，证书和证书的PKCS＃11令牌。

PKCS＃11令牌需要在JVM级别进行其他配置，并且可以通过以下设置启用：

* xpack.monitoring.exporters.$NAME.keystore.type
  
  * 将其设置PKCS11为表示PKCS＃11令牌应该用作密钥库。
* xpack.monitoring.exporters.$NAME.truststore.type
  * 将其设置PKCS11为表示PKCS＃11令牌应该用作信任库。
  > 在配置PKCS你的JVM被配置为一个密钥或Elasticsearch一个信任使用＃11令牌，该令牌PIN码可以通过适当的值设置进行配置ssl.truststore.password 或ssl.truststore.secure_password在所配置的上下文。由于只能配置一个PKCS＃11令牌，因此在Elasticsearch中只能使用一个密钥库和信任库进行配置。这反过来意味着在传输和http层中只有一个证书可用于TLS。

### 安全设定 xpack

默认情况下，当您拥有基本或试用许可证时，将禁用Elasticsearch安全功能。要启用安全功能，请使用该xpack.security.enabled 设置。

您配置xpack.security设置以 启用匿名访问 和执行消息身份验证， 设置文档和字段级别安全性， 配置域， 加密与SSL的通信以及 审核安全事件。

所有这些设置都可以添加到elasticsearch.yml配置文件中，但您添加到Elasticsearch密钥库的安全设置除外。有关创建和更新Elasticsearch密钥库的详细信息，请参阅 安全设置。

#### 常规安全设置

* xpack.security.enabled
  * 设置为true在节点上启用Elasticsearch安全功能。
  * 如果设置false为（基本和试用许可证的默认值），则会禁用安全功能。它还会影响连接到此Elasticsearch实例的所有Kibana实例; 您无需在这些kibana.yml文件中禁用安全功能。有关在特定Kibana实例中禁用安全功能的详细信息，请参阅 Kibana安全设置。
  > 如果您拥有金牌或更高牌照，则默认值为true; 我们建议您明确添加此设置以避免混淆。
* xpack.security.hide_settings
  
  * 从群集节点信息API的结果中省略的逗号分隔的设置列表 。您可以使用通配符在列表中包含多个设置。例如，以下值隐藏了ad1 active_directory域的所有设置： xpack.security.authc.realms.active_directory.ad1.*。API已经省略了所有ssl设置，bind_dn并且bind_password由于信息的敏感性。
* xpack.security.fips_mode.enabled
  
  * 启用fips操作模式。true如果在启用FIPS 140-2的JVM中运行此Elasticsearch实例，请将此设置为。有关更多信息，请参阅FIPS 140-2。默认为false。

#### 默认密码安全设置

* xpack.security.authc.accept_default_password
  * 在elasticsearch.yml，设置此项以false禁用对默认“changeme”密码的支持。
  * 密码哈希设置编辑
* xpack.security.authc.password_hashing.algorithm
  * 指定用于安全用户凭据存储的散列算法。请参阅表2“密码哈希算法”。默认为bcrypt。
  * 匿名访问设置编辑
  * 您可以在中配置以下匿名访问设置 elasticsearch.yml。有关更多信息，请参阅 启用匿名访问。
* xpack.security.authc.anonymous.username
  * 匿名用户的用户名（主体）。默认为_es_anonymous_user。
* xpack.security.authc.anonymous.roles
  * 与匿名用户关联的角色。需要。
* xpack.security.authc.anonymous.authz_exception
  * 何时true，如果匿名用户没有对请求的操作具有适当的权限，则返回HTTP 403响应。系统不会提示用户提供访问所请求资源的凭据。设置false为时，将返回HTTP 401响应，用户可以提供具有相应权限的凭据以获取访问权限。默认为true。

#### 自动设置

在安全功能接受通配符模式的位置（例如，角色中的索引模式，角色映射API中的组匹配），每个模式都被编译为自动机。以下设置可用于控制此行为。

* xpack.security.automata.max_determinized_states
  * 单个模式可以创建多少自动机状态的上限。这可以防止太难（例如指数级硬）的模式。默认为100,000。
* xpack.security.automata.cache.enabled
  * 是否缓存已编译的自动机。编译自动机可能是CPU密集型的，可能会减慢某些操作。缓存降低了自动机需要编译的频率。默认为true。
* xpack.security.automata.cache.size
  * 自动机缓存中保留的最大项目数。默认为10,000。
* xpack.security.automata.cache.ttl
  * 保留在自动机缓存中的项目中的时间长度（基于最近的用法）。默认为48h（48小时）。
  * 文档和字段级安全设置编辑
  * 您可以在中设置以下文档和字段级安全设置elasticsearch.yml。有关更多信息，请参阅 设置文档和字段级安全性。
* xpack.security.dls_fls.enabled
  * 设置为false阻止配置文档和字段级安全性。默认为true。

#### 令牌服务设置编辑
您可以在中设置以下令牌服务设置 elasticsearch.yml。

* xpack.security.authc.token.enabled
  * 设置为false禁用内置令牌服务。默认为true，除非 xpack.security.http.ssl.enabled是false。这可以防止通过普通http从连接中嗅探令牌。
* xpack.security.authc.token.timeout
  * 令牌有效的时间长度。默认情况下，此值为20m20分钟。最大值为1小时。
  * API密钥服务设置编辑
  * 您可以在其中设置以下API密钥服务设置 elasticsearch.yml。
* xpack.security.authc.api_key.enabled
  * 设置为false禁用内置API密钥服务。默认为true，除非 xpack.security.http.ssl.enabled是false。这可以防止通过普通http从连接嗅探API密钥。
* xpack.security.authc.api_key.hashing.algorithm
  * 指定用于保护API密钥凭据的散列算法。请参阅表2“密码哈希算法”。默认为pbkdf2。
* xpack.security.authc.api_key.cache.ttl
  * 缓存的API密钥条目的生存时间。API密钥ID和其API密钥的哈希在此段时间内被缓存。使用标准Elasticsearch 时间单位指定时间段。默认为1d。
* xpack.security.authc.api_key.cache.max_keys
  * 在任何给定时间可以在缓存中存在的最大API密钥条目数。默认为10,000。
* xpack.security.authc.api_key.cache.hash_algo
  * （专家设置）用于内存缓存API密钥凭证的散列算法。有关可能的值，请参见表1“缓存哈希算法”。默认为ssha256。

#### 领域设置

您在xpack.security.authc.realms 命名空间中配置域设置elasticsearch.yml。例如：

```yaml
xpack.security.authc.realms:

    native.realm1:
        order: 0
        ...

    ldap.realm2:
        order: 1
        ...

    active_directory.realm3:
        order: 2
        ...
    ...
```

##### 设置对所有领域

* type
  * 该类型的境界：native, `ldap，active_directory，pki，或file。需要。
* order
  * 领域内领域的优先级。首先咨询具有较低订单的领域。虽然不是必需的，但强烈建议在配置多个域时使用此设置。默认为Integer.MAX_VALUE。
* enabled
  * 指示是否启用了域。您可以使用此设置禁用领域而不删除其配置信息。默认为true。

##### 本机领域设置

对于本机领域，type必须设置为native。除了对所有领域有效的 设置外，您还可以指定以下可选设置：

* cache.ttl
  * 缓存用户条目的生存时间。在此段时间内缓存用户及其凭据的哈希值。使用标准Elasticsearch 时间单位指定时间段。默认为20m。
* cache.max_users
  * 在任何给定时间可以在缓存中存在的最大用户条目数。默认为100,000。
* cache.hash_algo
  * （专家设置）用于内存缓存用户凭据的散列算法。有关可能的值，请参见表1“缓存哈希算法”。默认为ssha256。
* authentication.enabled
  * 如果设置为false，则禁用此领域中的身份验证支持，以便它仅支持用户查找。（请参阅run as和 authorization领域功能）。默认为true。

##### 文件领域设置

该type设置必须设置为file。除了对所有领域有效的 设置外，您还可以指定以下设置：

* cache.ttl
  * 缓存用户条目的生存时间。在此配置的时间段内缓存用户及其凭据的哈希值。默认为20m。使用标准Elasticsearch 时间单位指定值。默认为20m。
* cache.max_users
  * 在给定时间可以在缓存中存在的最大用户条目数。默认为100,000。
* cache.hash_algo
  * （专家设置）用于内存缓存用户凭据的散列算法。请参见表1“缓存哈希算法”。默认为ssha256。
* authentication.enabled
  * 如果设置为false，则禁用此领域中的身份验证支持，以便它仅支持用户查找。（请参阅run as和 authorization领域功能）。默认为true。

##### LDAP领域设置编辑

该type设置必须设置为ldap。除了对所有领域编辑有效的 设置外，您还可以指定以下设置：

* url
  * ldap[s]://<server>:<port>格式 中的一个或多个LDAP URL 。需要。
  * 要提供多个URL，请使用YAML数组（["ldap://server1:636", "ldap://server2:636"]）或逗号分隔的字符串（"ldap://server1:636, ldap://server2:636"）。
  * 虽然两者都受支持，但您无法混合使用ldap和ldaps协议。
* load_balance.type
  
  * 定义多个LDAP URL时要使用的行为。有关支持的值，请参阅负载平衡和故障转移类型。默认为failover。
* load_balance.cache_ttl
  
  * 使用dns_failover或dns_round_robin作为负载平衡类型时，此设置控制缓存DNS查找的时间量。默认为1h。
* bind_dn
  
  * 用于绑定到LDAP并执行搜索的用户的DN。仅适用于用户搜索模式。如果未指定，则尝试匿名绑定。默认为空。由于其潜在的安全影响，bind_dn不会通过节点信息API公开。
* bind_password
  
  * [ 6.3 ] 在6.3中弃用。 请secure_bind_password改用。用于绑定到LDAP目录的用户的密码。默认为空。由于其潜在的安全影响，bind_password不会通过节点信息API公开。
* secure_bind_password（安全）
  
  * 用于绑定到LDAP目录的用户的密码。默认为空。
* user_dn_templates
  
  * 使用字符串替换用户名的DN模板{0}。此设置为多值; 您可以指定多个用户上下文。需要在用户模板模式下运行。如果user_search.base_dn指定，则此设置无效。有关不同模式的更多信息，请参阅LDAP领域。
* authorization_realms
  * 应授权委托授权的领域名称。如果使用此设置，则LDAP领域不执行角色映射，而是从列出的领域加载用户。引用的域按照它们在此列表中定义的顺序进行查阅。请参阅将授权委派给另一个领域
  > 如果user_search指定了任何开始的 user_dn_templates设置，则忽略设置。
* user_group_attribute
  
  * 指定要在用户上检查组成员身份的属性。如果group_search指定了任何设置，则忽略此设置。默认为memberOf。
* user_search.base_dn
  
  * 指定用于搜索用户的容器DN。需要在用户搜索模式下操作。如果user_dn_templates指定，则此设置无效。有关不同模式的更多信息，请参阅LDAP领域。
* user_search.scope
  
  * 用户搜索的范围。有效值sub_tree，one_level或 base。one_level只搜索直接包含在其中的对象 base_dn。sub_tree搜索下面包含的所有对象base_dn。 base指定base_dn是用户对象，并且它是唯一考虑的用户。默认为 sub_tree。
* user_search.filter
  
  * 指定用于搜索目录的过滤器，尝试将条目与用户提供的用户名进行匹配。默认为(uid={0})。 {0}用搜索时提供的用户名替换。
* user_search.attribute
  
  * [ 5.6 ] 在5.6中弃用。 请user_search.filter改用。与请求一起发送的用户名匹配的属性。默认为uid。
* user_search.pool.enabled
  
  * 启用或禁用用户搜索的连接池。如果设置为false，则为每个搜索创建一个新连接。默认设置为truewhen bind_dn。
* user_search.pool.size
  
  * 连接池中允许的LDAP服务器的最大连接数。默认为20。
* user_search.pool.initial_size
  
  * 启动时要创建到LDAP服务器的初始连接数。默认为0。如果LDAP服务器关闭，则值大于0可能导致启动失败。
* user_search.pool.health_check.enabled
  
  * 启用或禁用连接池中LDAP连接的运行状况检查。按指定的时间间隔在后台检查连接。默认为true。
* user_search.pool.health_check.dn
  
  * 作为运行状况检查的一部分检索的可分辨名称。默认值为bind_dnif present; 如果没有，请回到user_search.base_dn。
* user_search.pool.health_check.interval
  
  * 执行池中连接的后台检查的时间间隔。默认为60s。
* group_search.base_dn
  
  * 用于搜索用户具有成员资格的组的容器DN。如果此元素不存在，Elasticsearch user_group_attribute将在用户上搜索set 指定的属性 ，以确定组成员身份。
* group_search.scope
  
  * 指定组搜索是否应该是sub_tree，one_level或 base。 one_level只搜索直接包含在其中的对象 base_dn。sub_tree搜索下面包含的所有对象base_dn。 base指定它base_dn是一个组对象，并且它是唯一考虑的组。默认为 sub_tree。
* group_search.filter
  
  * 指定用于查找组的过滤器。如果没有设置，境界搜索group，groupOfNames，groupOfUniqueNames，或posixGroup与属性member，memberOf或memberUid。{0}过滤器中的任何实例都将替换为中定义的用户属性 group_search.user_attribute。
* group_search.user_attribute
  
  * 指定作为过滤器参数提取和提供的用户属性。如果未设置，则将用户DN传递到过滤器。默认为空。
* unmapped_groups_as_roles
  
  * 如果设置为true，则任何未映射的LDAP组的名称将用作角色名称并分配给用户。如果组未在角色映射文件中引用， 则认为该组未映射。不考虑基于API的角色映射。默认为false。
* files.role_mapping
  
  * 该位置为 YAML角色映射配置文件。默认为 ES_PATH_CONF/role_mapping.yml。
* follow_referrals
  
  * 指定Elasticsearch是否应遵循LDAP服务器返回的引用。引用是服务器返回的用于继续LDAP操作的URL（例如，搜索）。默认为true。
* metadata
  
  * 应从LDAP服务器加载并存储在经过身份验证的用户的元数据字段中的其他LDAP属性的列表。
* timeout.tcp_connect
  
  * 用于建立LDAP连接的TCP连接超时时间。最后一个s表示秒，或ms表示毫秒。默认为5s（5秒）。
* timeout.tcp_read
  
  * 建立LDAP连接后的TCP读取超时时间。最后一个s表示秒，或ms表示毫秒。默认为5s（5秒）。
* timeout.ldap_search
  
  * LDAP服务器强制执行LDAP搜索的超时期限。最后一个s表示秒，或ms表示毫秒。默认为5s（5秒）。
* ssl.key
  
  * 包含私钥的PEM编码文件的路径，如果LDAP服务器需要客户端身份验证，则使用该私钥。ssl.key并且ssl.keystore.path 不能同时使用。
* ssl.key_passphrase
  
  * 用于解密私钥的密码。此值是可选的，因为密钥可能未加密。
* ssl.secure_key_passphrase（安全）
  
  * 用于解密私钥的密码。
* ssl.certificate
  
  * PEM编码文件的路径，该文件包含将在连接时呈现给客户端的证书（或证书链）。
* ssl.certificate_authorities
  
  * 应信任的PEM编码证书文件的路径列表。 ssl.certificate_authorities并且ssl.truststore.path不能同时使用。
* ssl.keystore.path
  
  * 包含私钥和证书的Java Keystore文件的路径。 ssl.key并且ssl.keystore.path可能不会同时使用。
* ssl.keystore.type
  
  * 密钥库文件的格式。应该jks使用Java Keystore格式，PKCS12使用PKCS＃12文件，或PKCS11使用PKCS＃11令牌。默认是jks。
* ssl.keystore.password
  
  * 密钥库的密码。
* ssl.keystore.secure_password（安全）
  
  * 密钥库的密码。
* ssl.keystore.key_password
  
  * 密钥库中密钥的密码。默认为密钥库密码。
* ssl.keystore.secure_key_password
  
  * 密钥库中密钥的密码。默认为密钥库密码。
* ssl.truststore.path
  
  * 包含要信任的证书的Java Keystore文件的路径。 ssl.certificate_authorities并且ssl.truststore.path不能同时使用。
* ssl.truststore.password
  
  * 信任库的密码。
* ssl.truststore.secure_password（安全）
  
  * 信任库的密码。
* ssl.truststore.type
  
  * 密钥库文件的格式。应该jks使用Java Keystore格式，PKCS12使用PKCS＃12文件，或PKCS11使用PKCS＃11令牌。默认是jks。
* ssl.verification_mode
  * 表示ldaps在中间攻击和证书伪造时用于防范人员的验证类型。值是none，certificate和full。默认为full。
  * 有关ssl.verification_mode这些值的解释，请参阅。
* ssl.supported_protocols
  
  * 支持的TLS / SSL协议（带版本）。TLSv1.3,TLSv1.2,TLSv1.1如果JVM支持TLSv1.3，则默认为，否则为TLSv1.2,TLSv1.1。
* ssl.cipher_suites
  
  * 指定与LDAP服务器通信时应支持的密码套件。支持的密码套件可以在Oracle的 Java Cryptography Architecture文档中找到。请参阅ssl.cipher_suites 默认值。
* cache.ttl
  
  * 指定缓存用户条目的生存时间。在此段时间内缓存用户及其凭据的哈希值。使用标准的Elasticsearch 时间单位。默认为 20m。
* cache.max_users
  
  * 指定缓存可以包含的最大用户条目数。默认为100000。
* cache.hash_algo
  
  * （专家设置）指定用于内存缓存用户凭据的散列算法。请参见表1“缓存哈希算法”。默认为ssha256。
* authentication.enabled
  
  * 如果设置为false，则禁用此领域中的身份验证支持，以便它仅支持用户查找。（请参阅run as和 authorization领域功能）。默认为true。

#### Active Directory域设置

windows 的域控制, 过

[active direcotry](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html#ref-ad-settings)


#### PKI 认证设置

[pki 认证](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html#ref-pki-settings)

#### SAML 认证设置

[saml](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html#ref-saml-settings)

#### SAML 签名设置

[saml签名设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html#ref-saml-signing-settings)

#### SAML认证加密设置

[saml认证加密设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html#ref-saml-encryption-settings)

#### SAML 认证ssl设置

[SAML 认证ssl设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html#ref-saml-ssl-settings)

#### Kerberos 认证设置

[Kerberos 认证设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html#ref-kerberos-settings)

#### 负载平衡和故障转移编辑

该load_balance.type设置可以具有以下值：

* failover：指定的URL按指定的顺序使用。可以连接的第一台服务器将用于所有后续连接。如果与该服务器的连接失败，则可以建立连接的下一个服务器将用于后续连接。
* dns_failover：在此操作模式下，只能指定一个URL。此URL必须包含DNS名称。将查询系统以查找与此DNS名称对应的所有IP地址。始终会按照检索它们的顺序尝试与Active Directory或LDAP服务器的连接。这不同之处在于failover没有列表的重新排序，并且如果服务器在列表的开头出现故障，则仍将针对每个后续连接尝试它。
* round_robin：Connections将不断遍历提供的URL列表。如果服务器不可用，则迭代URL列表将继续，直到成功建立连接。
* dns_round_robin：在此操作模式下，只能指定一个URL。此URL必须包含DNS名称。将查询系统以查找与此DNS名称对应的所有IP地址。连接将不断迭代地址列表。如果服务器不可用，则迭代URL列表将继续，直到成功建立连接。

#### TLS / SSL设置的默认值编辑

通常，以下值表示各种TLS设置的默认值。有关更多信息，请参阅 加密通信。

* ssl.supported_protocols
  * 支持的版本协议。有效协议：SSLv2Hello， SSLv3，TLSv1，TLSv1.1，TLSv1.2，TLSv1.3。TLSv1.3,TLSv1.2,TLSv1.1如果JVM支持TLSv1.3，则默认为，否则为TLSv1.2,TLSv1.1。
  > 如果xpack.security.fips_mode.enabled是true，你不能使用SSLv2Hello 或SSLv3。见FIPS 140-2。
* ssl.client_authentication
  
  * 控制服务器关于从客户端连接请求证书的行为。有效值是required，optional和none。 required强制客户端提供证书，同时optional 请求客户端证书，但客户端不需要提供证书。默认为required，HTTP除外，默认为none。请参阅 HTTP TLS / SSL设置。
* ssl.verification_mode
  * 控制证书的验证。有效值为：
  * full，它验证提供的证书是否由受信任的颁发机构（CA）签名，并验证服务器的主机名（或IP地址）是否与证书中标识的名称匹配。
  * certificate，它验证提供的证书是否由受信任的颁发机构（CA）签名，但不执行任何主机名验证。
  * none，执行不验证服务器的证书。此模式禁用SSL / TLS的许多安全优势，只应在仔细考虑后使用。它主要用作尝试解决TLS错误时的临时诊断机制，强烈建议不要在生产群集上使用它。
  * 默认值为full。
* ssl.cipher_suites
  
  * 支持的密码套件可以在Oracle的 Java Cryptography Architecture文档中找到。默认为TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256， TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256，TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA，TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA， TLS_RSA_WITH_AES_128_CBC_SHA256，TLS_RSA_WITH_AES_128_CBC_SHA。如果Java加密扩展（JCE）无限强度权限策略文件已安装，则默认值还包括TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384， TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384，TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA，TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA， TLS_RSA_WITH_AES_256_CBC_SHA256，TLS_RSA_WITH_AES_256_CBC_SHA。

##### TLS / SSL密钥和可信证书设置编辑

以下设置用于指定在通过SSL / TLS连接进行通信时应使用的私钥，证书和可信证书。如果未配置受信任证书，则JVM信任的默认证书以及与同一上下文中的密钥关联的证书将受到信任。对于需要客户端身份验证的连接或充当启用SSL的服务器的连接，必须具有密钥和证书。

> 将受信任的证书存储在PKCS＃12文件中虽然受到支持，但在实践中并不常见。该elasticsearch-certutil工具以及Java keytool用于生成PKCS＃12文件，这些文件既可以用作密钥库，也可以用作信任库，但对于使用其他工具创建的容器文件可能不是这种情况。通常，PKCS＃12文件仅包含秘密和私人条目。要确认一个PKCS＃12容器包括受信任的证书（“锚”）条目寻找 2.16.840.1.113894.746875.1.1: <Unsupported tag 6>在 openssl pkcs12 -info输出，或trustedCertEntry在 keytool -list输出。

#### HTTP TLS / SSL设置
j
您可以配置以下TLS / SSL设置。如果未配置设置， 则使用默认TLS / SSL设置。

* xpack.security.http.ssl.enabled
  * 用于启用或禁用TLS / SSL。默认是false。
* xpack.security.http.ssl.supported_protocols
  * 支持的版本协议。有效协议：SSLv2Hello， SSLv3，TLSv1，TLSv1.1，TLSv1.2，TLSv1.3。TLSv1.3,TLSv1.2,TLSv1.1如果JVM支持TLSv1.3，则默认为，否则为TLSv1.2,TLSv1.1。
* xpack.security.http.ssl.client_authentication
  * 控制服务器关于从客户端连接请求证书的行为。有效值是required，optional和none。 required强制客户端提供证书，同时optional 请求客户端证书，但客户端不需要提供证书。默认为none。
* xpack.security.http.ssl.cipher_suites
  * 支持的密码套件可以在Oracle的 Java Cryptography Architecture文档中找到。默认为``。

#### HTTP TLS / SSL密钥和可信证书设置

以下设置用于指定在通过SSL / TLS连接进行通信时应使用的私钥，证书和可信证书。必须配置私钥和证书。如果未指定以下任何设置，则使用默认TLS / SSL设置。

PEM编码文件编辑

使用PEM编码文件时，请使用以下设置：

* xpack.security.http.ssl.key
  * 包含私钥的PEM编码文件的路径。
* xpack.security.http.ssl.key_passphrase
  * 用于解密私钥的密码。此值是可选的，因为密钥可能未加密。
* xpack.security.http.ssl.secure_key_passphrase（安全）
  * 用于解密私钥的密码。此值是可选的，因为密钥可能未加密。
* xpack.security.http.ssl.certificate
  * PEM编码文件的路径，其中包含将在请求时显示的证书（或证书链）。
* xpack.security.http.ssl.certificate_authorities
  * 应受信任的PEM编码证书文件的路径列表。

#### Java密钥库文件

使用包含应信任的私钥，证书和证书的Java密钥库文件（JKS）时，请使用以下设置：

* xpack.security.http.ssl.keystore.path
  * 保存私钥和证书的密钥库的路径。
* xpack.security.http.ssl.keystore.password
  * 密钥库的密码。
* xpack.security.http.ssl.keystore.secure_password（安全）
  * 密钥库的密码。
* xpack.security.http.ssl.keystore.key_password
  * 密钥库中私钥的密码。默认值为xpack.security.http.ssl.keystore.password。
* xpack.security.http.ssl.keystore.secure_key_password（安全）
  * 密钥库中私钥的密码。
* xpack.security.http.ssl.truststore.path
  * 信任库文件的路径。
* xpack.security.http.ssl.truststore.password
  * 信任库的密码。
* xpack.security.http.ssl.truststore.secure_password（安全）
  * 信任库的密码。

#### PKCS＃12文件

可以将Elasticsearch配置为使用包含应受信任的私钥，证书和证书的PKCS＃12容器文件（.p12或.pfx文件）。

PKCS＃12文件的配置方式与Java Keystore Files相同：

* xpack.security.transport.ssl.keystore.path
  * 保存私钥和证书的PKCS＃12文件的路径。
* xpack.security.transport.ssl.keystore.type
  * 将其设置PKCS12为表示密钥库是PKCS＃12文件。
* xpack.security.transport.ssl.keystore.password
  * PKCS＃12文件的密码。
* xpack.security.transport.ssl.keystore.secure_password（安全）
  * PKCS＃12文件的密码。
* xpack.security.transport.ssl.keystore.key_password
  * 存储在PKCS＃12文件中的私钥的密码。默认值为xpack.security.transport.ssl.keystore.password。
* xpack.security.transport.ssl.keystore.secure_key_password（安全）
  * 存储在PKCS＃12文件中的私钥的密码。
* xpack.security.transport.ssl.truststore.path
  * PKCS＃12文件的路径，用于保存要信任的证书。
* xpack.security.transport.ssl.truststore.type
  * 将其设置PKCS12为表示信任库是PKCS＃12文件。
* xpack.security.transport.ssl.truststore.password
  * PKCS＃12文件的密码。
* xpack.security.transport.ssl.truststore.secure_password（安全）
  * PKCS＃12文件的密码。

#### PKCS＃11令牌

 可以将Elasticsearch配置为使用包含应受信任的私钥，证书和证书的PKCS＃11令牌。

PKCS＃11令牌需要在JVM级别进行其他配置，并且可以通过以下设置启用：

* xpack.security.transport.keystore.type
  
  * 将其设置PKCS11为表示PKCS＃11令牌应该用作密钥库。
* xpack.security.transport.truststore.type
  * 将其设置PKCS11为表示PKCS＃11令牌应该用作信任库。
  > 在配置PKCS你的JVM被配置为一个密钥或Elasticsearch一个信任使用＃11令牌，该令牌PIN码可以通过适当的值设置进行配置ssl.truststore.password 或ssl.truststore.secure_password在所配置的上下文。由于只能配置一个PKCS＃11令牌，因此在Elasticsearch中只能使用一个密钥库和信任库进行配置。这反过来意味着在传输和http层中只有一个证书可用于TLS。

#### 传输配置文件TLS / SSL设置

每个传输配置文件也可以使用与默认传输相同的设置。默认情况下，除非指定传输配置文件，否则传输配置文件的设置将与默认传输相同。

例如，让我们看一下关键设置。对于默认传输，这是xpack.security.transport.ssl.key。要在传输配置文件中使用此设置，请使用前缀transport.profiles.$PROFILE.xpack.security.并在之后附加设置的部分xpack.security.transport.。对于键设置，这将是transport.profiles.$PROFILE.xpack.security.ssl.key。

#### IP过滤设置

您可以配置以下IP过滤设置。

* xpack.security.transport.filter.allow
  * 允许的IP地址列表。
* xpack.security.transport.filter.deny
  * 拒绝的IP地址列表。
* xpack.security.http.filter.allow
  * 仅允许HTTP的IP地址列表。
* xpack.security.http.filter.deny
  * 仅拒绝HTTP的IP地址列表。
* transport.profiles.$PROFILE.xpack.security.filter.allow
  * 允许此配置文件的IP地址列表。
* transport.profiles.$PROFILE.xpack.security.filter.deny
  * 要拒绝此配置文件的IP地址列表。

#### 用户缓存和密码哈希算法

某些领域将用户凭据存储在内存中。为了限制凭证被盗的风险并减轻凭据泄露，缓存仅在用户凭证中存储散列版本的用户凭证。默认情况下，使用salted sha-256哈希算法对用户缓存进行哈希处理。您可以通过将cache.hash_algo领域设置设置为以下任何值来使用不同的哈希算法：

表1.缓存哈希算法

| 算法 | 描述 |
| ------ | ------ |
| ssha256 | 使用盐渍sha-256算法（默认）。 |
| md5 | 使用MD5算法。 |
| sha1 | 使用SHA1算法。 |
| bcrypt | 使用bcrypt1024轮生成的盐算法。 |
| bcrypt4 | 使用bcrypt16轮生成的盐算法。 |
| bcrypt5 | 使用bcrypt32轮生成的盐算法。 |
| bcrypt6 | 使用bcrypt64轮生成的盐算法。 |
| bcrypt7 | 使用bcrypt在128轮中生成的盐的算法。 |
| bcrypt8 | 使用bcrypt256轮生成的盐算法。 |
| bcrypt9 | 使用bcrypt512轮生成的盐算法。 |
| pbkdf2 | 使用PBKDF2密钥导出函数HMAC-SHA512作为使用10000次迭代的伪随机函数。 |
| pbkdf2_1000 | 使用PBKDF2密钥导出函数HMAC-SHA512作为伪随机函数，使用1000次迭代。 |
| pbkdf2_10000 | 使用PBKDF2密钥导出函数HMAC-SHA512作为使用10000次迭代的伪随机函数。 |
| pbkdf2_50000 | 使用PBKDF2密钥导出函数HMAC-SHA512作为使用50000次迭代的伪随机函数。 |
| pbkdf2_100000 | 使用PBKDF2密钥导出函数HMAC-SHA512作为使用100000次迭代的伪随机函数。 |
| pbkdf2_500000 | 使用PBKDF2密钥导出函数HMAC-SHA512作为使用500000次迭代的伪随机函数。 |
| pbkdf2_1000000 | 使用PBKDF2密钥导出函数HMAC-SHA512作为使用1000000次迭代的伪随机函数。 |
| noop，clear_text | 不散列凭据并将其保留在内存中的明文中。注意：保持明文不被认为是不安全的，并且可能在操作系统级别受到损害（例如通过内存转储和使用ptrace）。 |


同样，存储密码的领域使用加密强和密码特定的salt值对它们进行哈希处理。您可以通过将xpack.security.authc.password_hashing.algorithm设置设置为以下之一来配置密码哈希算法：

表2.密码散列算法

| 算法 | 描述 |
| ------ | ------ |
| bcrypt | 使用bcrypt1024轮生成的盐算法。（默认） |
| bcrypt4 | 使用bcrypt16轮生成的盐算法。 |
| bcrypt5 | 使用bcrypt32轮生成的盐算法。 |
| bcrypt6 | 使用bcrypt64轮生成的盐算法。 |
| bcrypt7 | 使用bcrypt在128轮中生成的盐的算法。 |
| bcrypt8 | 使用bcrypt256轮生成的盐算法。 |
| bcrypt9 | 使用bcrypt512轮生成的盐算法。 |
| bcrypt10 | 使用bcrypt1024轮生成的盐算法。 |
| bcrypt11 | 使用bcrypt2048轮生成的盐算法。 |
| bcrypt12 | 使用bcrypt4096轮生成的盐算法。 |
| bcrypt13 | 使用bcrypt8192轮生成的盐算法。 |
| bcrypt14 | 使用bcrypt16384轮生成的盐算法。 |
| pbkdf2 | 使用PBKDF2密钥导出函数HMAC-SHA512作为使用10000次迭代的伪随机函数。 |
| pbkdf2_1000 | 使用PBKDF2密钥导出函数HMAC-SHA512作为伪随机函数，使用1000次迭代。 |
| pbkdf2_10000 | 使用PBKDF2密钥导出函数HMAC-SHA512作为使用10000次迭代的伪随机函数。 |
| pbkdf2_50000 | 使用PBKDF2密钥导出函数HMAC-SHA512作为使用50000次迭代的伪随机函数。 |
| pbkdf2_100000 | 使用PBKDF2密钥导出函数HMAC-SHA512作为使用100000次迭代的伪随机函数。 |
| pbkdf2_500000 | 使用PBKDF2密钥导出函数HMAC-SHA512作为使用500000次迭代的伪随机函数。 |
| pbkdf2_1000000 | 使用PBKDF2密钥导出函数HMAC-SHA512作为使用1000000次迭代的伪随机函数。 |

### SQL访问设置 xpark

默认情况下启用SQL Access。您可以在elasticsearch.yml文件中配置这些SQL Access设置。

#### 常规SQL Access设置编辑

* xpack.sql.enabled
  * 设置为false在节点上禁用SQL Access

### watcher 设置 xpark

您配置Watcher设置以设置Watcher并通过电子邮件， Slack和 PagerDuty发送通知 。

所有这些设置都可以添加到elasticsearch.yml配置文件中，但您添加到Elasticsearch密钥库的安全设置除外。有关创建和更新Elasticsearch密钥库的详细信息，请参阅 安全设置。还可以使用群集更新设置API在 群集中更新动态设置。

#### watcher 的一般设置

* xpack.watcher.enabled
  * 设置为false禁用节点上的Watcher。
* xpack.watcher.encrypt_sensitive_data
  * 设置为true加密敏感数据。如果启用此设置，则还必须指定xpack.watcher.encryption_key设置。有关更多信息，请参阅 在Watcher中加密敏感数据。
* xpack.watcher.encryption_key（安全）
  * 指定包含用于加密敏感数据的密钥的文件的路径。如果xpack.watcher.encrypt_sensitive_data设置为true，则需要此设置。有关更多信息，请参阅 在Watcher中加密敏感数据。
* xpack.watcher.history.cleaner_service.enabled
  * [ 6.3.0 ] 在6.3.0中添加。默认更改为true。 [ 7.0.0 ] 在7.0.0中已弃用。Watcher历史指数现在由watch-history-ilm-policyILM政策 管理
  * 设置为true（默认）以启用清洁服务。如果是此设置 true，则xpack.monitoring.enabled还必须将设置设置为true启用本地导出器。更清洁的服务.watcher-history*在确定它们是旧的时删除以前版本的Watcher索引（例如）。Watcher指数的持续时间由xpack.monitoring.history.duration设置决定 ，默认为7天。有关该设置的详细信息，请参阅监视设置。
* xpack.http.proxy.host
  * 指定用于连接HTTP服务的代理服务器的地址。
* xpack.http.proxy.port
  * 指定用于连接代理服务器的端口号。
* xpack.http.default_connection_timeout
  * 在启动连接时等待请求中止的最长时间。
* xpack.http.default_read_timeout
  * 在请求中止之前，两个数据包之间的最大不活动时间。
* xpack.http.max_response_size
  * 指定允许HTTP响应的最大大小，默认值为 10mb，最大可配置值为50mb。
* xpack.http.whitelist
  * 允许内部HTTP客户端连接的URL列表。此客户端用于HTTP输入，webhook，slack，pagerduty和jira操作。此设置可以动态更新。它默认* 允许一切。注意：如果配置此设置并且您正在使用其中一个slack / pagerduty操作，则必须确保相应的端点也列入白名单。

#### Watcher TLS / SSL设置编辑

您可以配置以下TLS / SSL设置。如果未配置设置， 则使用默认TLS / SSL设置。

* xpack.http.ssl.supported_protocols
  * 支持的版本协议。有效协议：SSLv2Hello， SSLv3，TLSv1，TLSv1.1，TLSv1.2，TLSv1.3。TLSv1.3,TLSv1.2,TLSv1.1如果JVM支持TLSv1.3，则默认为，否则为TLSv1.2,TLSv1.1。
* xpack.http.ssl.verification_mode
  * 控制证书的验证。有效值是none， certificate和full。默认为full。
* xpack.http.ssl.cipher_suites
  * 支持的密码套件可以在Oracle的 Java Cryptography Architecture文档中找到。默认为``。

#### Watcher TLS / SSL密钥和可信证书设置

以下设置用于指定在通过SSL / TLS连接进行通信时应使用的私钥，证书和可信证书。私钥和证书是可选的，如果服务器需要客户端身份验证进行PKI身份验证，则会使用私钥和证书。如果未指定以下任何设置，则使用默认TLS / SSL设置。

#### PEM编码文件

使用PEM编码文件时，请使用以下设置：

* xpack.http.ssl.key
  * 包含私钥的PEM编码文件的路径。
* xpack.http.ssl.key_passphrase
  * 用于解密私钥的密码。此值是可选的，因为密钥可能未加密。
* xpack.http.ssl.secure_key_passphrase（安全）
  * 用于解密私钥的密码。此值是可选的，因为密钥可能未加密。
* xpack.http.ssl.certificate
  * PEM编码文件的路径，其中包含将在请求时显示的证书（或证书链）。
* xpack.http.ssl.certificate_authorities
  * 应受信任的PEM编码证书文件的路径列表。

#### Java密钥库文件

使用包含应信任的私钥，证书和证书的Java密钥库文件（JKS）时，请使用以下设置：

* xpack.http.ssl.keystore.path
  * 保存私钥和证书的密钥库的路径。
* xpack.http.ssl.keystore.password
  * 密钥库的密码。
* xpack.http.ssl.keystore.secure_password（安全）
  * 密钥库的密码。
* xpack.http.ssl.keystore.key_password
  * 密钥库中私钥的密码。默认值为xpack.http.ssl.keystore.password。
* xpack.http.ssl.keystore.secure_key_password（安全）
  * 密钥库中私钥的密码。
* xpack.http.ssl.truststore.path
  * 信任库文件的路径。
* xpack.http.ssl.truststore.password
  * 信任库的密码。
* xpack.http.ssl.truststore.secure_password（安全）
  * 信任库的密码。

#### PKCS＃12文件

可以将Elasticsearch配置为使用包含应受信任的私钥，证书和证书的PKCS＃12容器文件（.p12或.pfx文件）。

PKCS＃12文件的配置方式与Java Keystore Files相同：

* xpack.http.ssl.keystore.path
  * 保存私钥和证书的PKCS＃12文件的路径。
* xpack.http.ssl.keystore.type
  * 将其设置PKCS12为表示密钥库是PKCS＃12文件。
* xpack.http.ssl.keystore.password
  * PKCS＃12文件的密码。
* xpack.http.ssl.keystore.secure_password（安全）
  * PKCS＃12文件的密码。
* xpack.http.ssl.keystore.key_password
  * 存储在PKCS＃12文件中的私钥的密码。默认值为xpack.http.ssl.keystore.password。
* xpack.http.ssl.keystore.secure_key_password（安全）
  * 存储在PKCS＃12文件中的私钥的密码。
* xpack.http.ssl.truststore.path
  * PKCS＃12文件的路径，用于保存要信任的证书。
* xpack.http.ssl.truststore.type
  * 将其设置PKCS12为表示信任库是PKCS＃12文件。
* xpack.http.ssl.truststore.password
  * PKCS＃12文件的密码。
* xpack.http.ssl.truststore.secure_password（安全）
  * PKCS＃12文件的密码。

#### PKCS＃11令牌编辑

可以将Elasticsearch配置为使用包含应受信任的私钥，证书和证书的PKCS＃11令牌。

PKCS＃11令牌需要在JVM级别进行其他配置，并且可以通过以下设置启用：

* xpack.http.keystore.type
  
  * 将其设置PKCS11为表示PKCS＃11令牌应该用作密钥库。
* xpack.http.truststore.type
  * 将其设置PKCS11为表示PKCS＃11令牌应该用作信任库。
  > 在配置PKCS你的JVM被配置为一个密钥或Elasticsearch一个信任使用＃11令牌，该令牌PIN码可以通过适当的值设置进行配置ssl.truststore.password 或ssl.truststore.secure_password在所配置的上下文。由于只能配置一个PKCS＃11令牌，因此在Elasticsearch中只能使用一个密钥库和信任库进行配置。这反过来意味着在传输和http层中只有一个证书可用于TLS。

#### 电子邮件通知设置

您可以在中配置以下电子邮件通知设置 elasticsearch.yml。有关通过电子邮件发送通知的详细信息，请参阅配置电子邮件。

* xpack.notification.email.account
  * 指定通过电子邮件发送通知的帐户信息。您可以指定以下电子邮件帐户属性：
  * profile（动态）
    * 用于构建从帐户发送的MIME邮件 的电子邮件配置文件。有效值：standard，gmail和 outlook。默认为standard。
  * email_defaults.*（动态）
    * 一组可选的电子邮件属性，用作从帐户发送的电子邮件的默认值。请参阅支持的属性的 电子邮件操作属性。
  * smtp.auth（动态）
    * 设置为true尝试使用AUTH命令对用户进行身份验证。默认为false。
  * smtp.host（动态）
    * 要连接的SMTP服务器。需要。
  * smtp.port（动态）
    * 要连接的SMTP服务器端口。默认为25。
  * smtp.user（动态）
    * SMTP的用户名。需要。
  * smtp.secure_password（安全）
    * 指定SMTP用户的密码。
  * smtp.starttls.enable（动态）
    * 设置为true允许使用该STARTTLS 命令（如果服务器支持）在发出任何登录命令之前将连接切换到受TLS保护的连接。请注意，必须配置适当的信任存储，以便客户端信任服务器的证书。默认为false。
  * smtp.starttls.required（动态）
    * 如果true，那么STARTTLS将是必需的。如果该命令失败，则连接将失败。默认为false。
  * smtp.ssl.trust（动态）
    * 假定为受信任且已禁用证书验证的SMTP服务器主机列表。如果设置为“*”，则所有主机都是可信的。如果设置为以空格分隔的主机列表，则这些主机是受信任的。否则，信任取决于服务器提供的证书。
  * smtp.timeout（动态）
    * 套接字读取超时。默认是两分钟。
  * smtp.connection_timeout（动态）
    * 套接字连接超时。默认是两分钟。
  * smtp.write_timeout（动态）
    * 套接字写入超时。默认是两分钟。
  * smtp.local_address（动态）
    * 发送电子邮件时可配置的本地地址。默认情况下未配置。
  * smtp.local_port（动态）
    * 发送电子邮件时可配置的本地端口。默认情况下未配置。
  * smtp.send_partial（动态）
    * 发送电子邮件，尽管其中一个接收方地址无效。
  * smtp.wait_on_quit（动态）
    * 如果设置为false，则发送QUIT命令并关闭连接。如果设置为true，则发送QUIT命令并等待回复。默认为True。
* xpack.notification.email.html.sanitization.allow
  * 指定电子邮件通知中允许的HTML元素。有关更多信息，请参阅配置HTML清理选项。您可以指定单个HTML元素和以下HTML功能组：
  * _tables
    * 表中的所有相关的元素：<table>，<th>，<tr> 和<td>。
  * _blocks
    * 以下块元素：<p>，<div>，<h1>， <h2>，<h3>，<h4>，<h5>，<h6>，<ul>，<ol>， <li>，和<blockquote>。
  * _formatting
    * 下面内嵌格式元素：<b>，<i>， <s>，<u>，<o>，<sup>，<sub>，<ins>，<del>， <strong>，<strike>，<tt>，<code>，<big>， <small>，<br>，<span>，和<em>。
  * _links
    * 的<a>与元件href：使用以下协议指向一个URL属性http，https 和mailto。
  * _styles
    * style所有元素 的属性。请注意，CSS属性也会被清理以防止XSS攻击。
  * img ， img:all
    * 所有图像（外部和嵌入式）。
  * img:embedded
    * 只有嵌入的图像。嵌入式图像只能cid:在其src属性中使用 URL协议。
* xpack.notification.email.html.sanitization.disallow
  * 指定电子邮件通知中不允许的HTML元素。您可以指定单个HTML元素和HTML要素组。
* xpack.notification.email.html.sanitization.enabled
  * 设置为false完全禁用HTML卫生。不建议。默认为true。

#### slack 通知设置

您可以在中配置以下 slack 通知设置 elasticsearch.yml。有关通过Slack发送通知的更多信息，请参阅 配置Slack。

* xpack.notification.slack
  * 指定通过Slack发送通知的帐户信息。您可以指定以下Slack帐户属性：
* secure_url（安全）
  * 用于将消息发布到Slack的传入Webhook URL。需要。
* message_defaults.from
  * 要在Slack消息中显示的发件人名称。默认为手表ID。
* message_defaults.to
  * 要发送消息的默认Slack通道或组。
* message_defaults.icon
  * 要在Slack消息中显示的图标。覆盖传入的webhook配置的图标。接受图像的公共URL。
* message_defaults.text
  * 默认邮件内容。
* message_defaults.attachment
  * 默认邮件附件。Slack消息附件使您可以创建更丰富格式的消息。指定为 Slack附件文档中定义的数组 。

#### Jira通知设置编辑

您可以在中配置以下Jira通知设置 elasticsearch.yml。有关使用通知在Jira中创建问题的更多信息，请参阅 配置Jira。

* xpack.notification.jira
  * 指定使用通知在Jira中创建问题的帐户信息。您可以指定以下Jira帐户属性：
  * secure_url（安全）
    * Jira Software服务器的URL。需要。
  * secure_user（安全）
    * 要连接到Jira Software服务器的用户的名称。需要。
  * secure_password（安全）
    * 用于连接Jira Software服务器的用户的密码。需要。
  * issue_defaults
    * Jira中创建的问题的默认字段值。有关更多信息，请参阅 Jira操作属性。可选的。

#### PagerDuty通知设置

您可以在中配置以下PagerDuty通知设置 elasticsearch.yml。有关通过PagerDuty发送通知的详细信息，请参阅 配置PagerDuty。

* xpack.notification.pagerduty
  * 指定通过PagerDuty发送通知的帐户信息。您可以指定以下PagerDuty帐户属性：
  * name
    * 与您用于访问PagerDuty的API密钥关联的PagerDuty帐户的名称。需要。
  * secure_service_api_key（安全）
    * 用于访问PagerDuty 的 PagerDuty API密钥。需要。
  * event_defaults
    * PagerDuty事件属性的 默认值 。可选的。
* description
  * 包含PagerDuty事件的默认描述的字符串。如果未配置默认值，则每个PagerDuty操作都必须指定a description。
* incident_key
  * 包含发送PagerDuty事件时要使用的默认事件键的字符串。
* client
  * 一个字符串，指定默认监视客户端。
* client_url
  * 默认监控客户端的URL。
* event_type
  * 默认事件类型。有效值：trigger，resolve，acknowledge。
* attach_payload
  * 是否默认将监视有效负载作为事件的上下文提供。有效值：true，false。

## 重要的Elasticsearch配置

虽然Elasticsearch只需要很少的配置，但在投入生产之前需要考虑许多设置。

在投入生产之前，必须考虑以下设置：

* 路径设置
* 群集名称
* 节点名称
* 网络主机
* 发现设置
* 堆大小
* 堆转储路径
* GC记录
* 临时目录

### path.data 和 path.logs

如果您使用.zip或.tar.gz存档，则data和logs 目录是子文件夹$ES_HOME。如果这些重要文件夹保留在其默认位置，则在将Elasticsearch升级到新版本时，存在删除它们的高风险。

在生产使用中，您几乎肯定会想要更改数据和日志文件夹的位置：

```yml
path:
  logs: /var/log/elasticsearch
  data: /var/data/elasticsearch
```

该RPM和Debian发行版已经使用自定义路径，data和logs。

该path.data设置可以被设置为多条路径，在这种情况下，所有的路径将被用于存储数据（虽然属于单个碎片文件将全部存储相同的数据路径上）：

```yml
path:
  data:
    - /mnt/elasticsearch_1
    - /mnt/elasticsearch_2
    - /mnt/elasticsearch_3
```

### cluster.name

节点只能cluster.name在与群集中的所有其他节点共享群集时才能加入群集。默认名称是elasticsearch，但您应将其更改为适当的名称，该名称描述了群集的用途。

`cluster.name: logging-prod`

确保不要在不同的环境中重用相同的群集名称，否则最终会导致节点加入错误的群集。

### node.name

Elasticsearch使用Elasticsearch node.name的特定实例作为人类可读标识符，因此它包含在许多API的响应中。它默认为Elasticsearch启动时机器具有的主机名，但可以elasticsearch.yml按如下方式显式配置 ：

`node.name:prod-data-2`

### network.host

默认情况下，Elasticsearch仅绑定到环回地址 - 例如127.0.0.1 和[::1]。这足以在服务器上运行单个开发节点。

> 实际上，可以从$ES_HOME 单个节点上的相同位置启动多个节点。这对于测试Elasticsearch形成集群的能力非常有用，但它不是推荐用于生产的配置。

为了在其他服务器上形成包含节点的集群，您的节点将需要绑定到非环回地址。虽然有许多 网络设置，但通常您需要配置的是 network.host：

`network.host:192.168.1.10`

该network.host设置也了解一些特殊的值，比如 _local_，_site_，_global_和喜欢修饰:ip4和:ip6，其中的细节中可以找到的特殊值network.host编辑。

> 只要您提供自定义设置network.host，Elasticsearch就会假定您正在从开发模式转移到生产模式，并将许多系统启动检查从警告升级到异常。有关详细信息，请参阅[开发模式与生产模式编辑](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html#dev-vs-prod)。

### 发现和群集形成设置

在开始生产之前，应该配置两个重要的发现和群集形成设置，以便群集中的节点可以相互发现并选择主节点。

#### discovery.seed_hosts编辑

开箱即用，没有任何网络配置，Elasticsearch将绑定到可用的环回地址，并将扫描本地端口9300到9305以尝试连接到在同一服务器上运行的其他节点。这提供了自动集群体验，无需进行任何配置。

如果要在其他主机上形成包含节点的群集，则必须使用该 discovery.seed_hosts设置提供群集中其他节点的列表，这些节点符合主要条件且可能是实时且可联系的，以便为发现过程设定种子。此设置通常应包含群集中所有符合主节点的节点的地址。此设置包含主机数组或逗号分隔的字符串。每个值应采用host:port或的形式host（如果未设置，则port 默认为设置transport.profiles.default.port回落 transport.port）。请注意，必须将IPv6主机置于括号内。此设置的默认值为127.0.0.1, [::1]。

#### cluster.initial_master_nodes编辑

当您第一次启动全新的Elasticsearch集群时，会出现一个集群引导步骤，该步骤确定在第一次选举中计票的主要合格节点集。在开发模式下，如果未配置发现设置，则此步骤由节点本身自动执行。由于此自动引导本质上是不安全的，因此当您在生产模式下启动全新集群时，必须明确列出符合条件的节点的名称或IP地址，这些节点的投票应在第一次选举中计算。使用该cluster.initial_master_nodes设置设置此列表。

```yml
discovery.seed_hosts:
   - 192.168.1.10:9300
   - 192.168.1.11  # 如果未指定 ，端口将默认为transport.profiles.default.port和回退 transport.port。
   - seeds.mydomain.com  # 如果主机名解析为多个IP地址，则该节点将尝试发现所有已解析地址的其他节点。
cluster.initial_master_nodes:
   - master-node-a  # 初始主节点可以通过它们来标识node.name，默认为主机名。确保值 cluster.initial_master_nodes与node.name确切匹配。如果使用完全限定的域名（例如master-node-a.example.com节点名称），则必须在此列表中使用完全限定名称; 相反，如果node.name是一个没有任何尾随限定符的裸主机名，那么你还必须省略尾随限定符 cluster.initial_master_nodes。
   - 192.168.1.12 # 初始主节点也可以通过其IP地址识别。
   - 192.168.1.13:9301  # 如果多个主节点共享一个IP地址，则必须使用传输端口来区分它们。
```

### 设置堆大小

默认情况下，Elasticsearch告诉JVM使用最小和最大大小为1 GB的堆。迁移到生产环境时，配置堆大小以确保Elasticsearch有足够的可用堆是很重要的。

Elasticsearch将通过（最小堆大小）和（最大堆大小）设置分配jvm.options中指定的整个堆 。XmsXmx

这些设置的值取决于服务器上可用的RAM量。好的经验法则是：

* 将最小堆大小（Xms）和最大堆大小（Xmx）设置为彼此相等。
* Elasticsearch可用的堆越多，它可用于缓存的内存就越多。但请注意，过多的堆可能会使您陷入长时间的垃圾收集暂停。
* 设置Xmx为不超过物理RAM的50％，以确保有足够的物理RAM用于内核文件系统缓存。
*  不要设置Xmx为JVM用于压缩对象指针（压缩oops）的截止值之上; 确切的截止值变化但接近32 GB。您可以通过查找日志中的行来验证您是否在限制之下，如下所示：
  * heap size [1.9gb], compressed ordinary object pointers [true]
* 更好的是，尽量保持低于零基础压缩oops的阈值; 确切的截止值有所不同，但大多数系统上26 GB是安全的，但在某些系统上可能高达30 GB。您可以通过使用JVM选项启动Elasticsearch -XX:+UnlockDiagnosticVMOptions -XX:+PrintCompressedOopsMode并查找如下所示的行来验证您是否在限制之下：
  * heap address: 0x000000011be00000, size: 27648 MB, zero based Compressed Oops
* 显示启用从零开始的压缩oops而不是
  * heap address: 0x0000000118400000, size: 28672 MB, Compressed Oops with base: 0x00000001183ff000

以下是如何通过jvm.options文件设置堆大小的示例：

```shell
-Xms2g # 将最小堆大小设置为2g。
-Xmx2g # 将最大堆大小设置为2g。
```

也可以通过环境变量设置堆大小。这可以通过注释掉来完成Xms，并Xmx设置在 jvm.options文件中，并通过设置这些值ES_JAVA_OPTS：

```shell
ES_JAVA_OPTS="-Xms2g -Xmx2g" ./bin/elasticsearch  # 将最小和最大堆大小设置为2 GB。
ES_JAVA_OPTS="-Xms4000m -Xmx4000m" ./bin/elasticsearch  # 将最小和最大堆大小设置为4000 MB。
```

> 为Windows服务配置堆不同于上面的堆。最初为Windows服务填充的值可以如上配置，但在安装服务后不同。有关其他详细信息，请参阅Windows服务文档。

### JVM堆转储路径 JVM heap dump path

默认情况下，Elasticsearch将JVM配置为将内存异常转储到默认数据目录（这 /var/lib/elasticsearch适用于RPM和Debian软件包发行版，以及data用于tar和zip归档文件分发的Elasticsearch安装根目录下的目录） 。如果这个路径是不适合接受堆转储，您应该修改的条目-XX:HeapDumpPath=...在 jvm.options。如果指定目录，JVM将根据正在运行的实例的PID为堆转储生成文件名。如果指定固定文件名而不是目录，则当JVM需要在内存不足异常上执行堆转储时，该文件不能存在，否则堆转储将失败。

### GC日志

默认情况下，Elasticsearch 启用GC日志。这些配置在 jvm.options默认位置和默认位置与 Elasticsearch 日志相同。默认配置每 64MB 轮换一次日志，最多可占用 2GB 的磁盘空间。

### 临时目录

默认情况下，Elasticsearch使用启动脚本在系统临时目录下创建的专用临时目录。

在某些Linux发行版上，系统实用程序将清除文件和目录（/tmp如果它们最近未被访问过）。如果长时间不使用需要临时目录的功能，则可能导致在Elasticsearch运行时删除专用临时目录。如果随后使用需要临时目录的功能，则会导致问题。

如果使用.deb或.rpm包安装Elasticsearch 并在其下运行，systemd那么Elasticsearch使用的专用临时目录将从定期清理中排除。

但是，如果您打算.tar.gz在Linux 上运行分发一段时间，那么您应该考虑为Elasticsearch创建一个专用的临时目录，该目录不在将从中清除旧文件和目录的路径下。此目录应具有权限集，以便只有运行Elasticsearch的用户才能访问它。然后$ES_TMPDIR在启动Elasticsearch之前将环境变量设置 为指向它。

### JVM致命错误日志 jvm fatal error

默认情况下，Elasticsearch 将 JVM 配置为将致命错误日志写入默认日志记录目录（这 /var/log/elasticsearch 适用于 RPM 和 Debian 软件包分发，以及 logs 针对 tar 和 zip 归档文件分发的 Elasticsearch 安装根目录下的目录 ）。这些是 JVM 在遇到致命错误（例如，分段(segmentation)错误）时生成的日志。如果该路径不适合于接收的日志，则应修改条目-XX:ErrorFile=...中 jvm.options到备用路径。

## 重要的系统设置

理想情况下，Elasticsearch应该在服务器上单独运行并使用它可用的所有资源。为此，您需要配置操作系统以允许运行Elasticsearch的用户访问比默认情况下允许的资源更多的资源。

在投入生产之前，必须考虑以下设置：

* 禁用交换
* 文件描述符
* 虚拟内存
* 线程数
* DNS缓存设置
* JNA临时目录未安装 noexec

默认情况下，Elasticsearch假定您正在开发模式下工作。如果未正确配置上述任何设置，则会向日志文件写入警告，但您将能够启动并运行Elasticsearch节点。

一旦配置了网络设置network.host，Elasticsearch就会假定您正在转向生产并将上述警告升级为异常。这些异常将阻止您的Elasticsearch节点启动。这是一项重要的安全措施，可确保您不会因服务器配置错误而丢失数据。

### 配置系统设置

配置系统设置的位置取决于您用于安装Elasticsearch的软件包以及您使用的操作系统。

使用.zip或.tar.gz包时，可以配置系统设置：

* 暂时用ulimit，或
* 永久地/etc/security/limits.conf。

使用RPM或Debian软件包时，大多数系统设置都在系统配置文件中设置 。但是，使用systemd的系统要求在systemd配置文件中指定系统限制 。

#### ulimit编辑

在Linux系统上，ulimit可以用于临时更改资源限制。通常需要root在切换到将运行Elasticsearch的用户之前设置限制。例如，要将打开文件句柄（ulimit -n）的数量设置为65,536，您可以执行以下操作：

```shell
sudo su  
ulimit -n 65535 ## 更改打开文件的最大数量。
su elasticsearch  ## 成为elasticsearch用户以启动Elasticsearch。
```

新限制仅在当前会话期间应用。

您可以查询所有当前应用的限制ulimit -a。

#### /etc/security/limits.conf编辑

在Linux系统上，可以通过编辑/etc/security/limits.conf文件为特定用户设置持久限制。要将用户的最大打开文件数设置elasticsearch为65,536，请将以下行添加到limits.conf文件中：

`elasticsearch  -  nofile  65535`

此更改仅在elasticsearch用户下次打开新会话时生效。

> Ubuntu和 limits.conf
> Ubuntu忽略limits.conf启动进程的文件init.d。要启用该limits.conf文件，请编辑/etc/pam.d/su并取消注释以下行：
> `##session required pam_limits.so`

#### Sysconfig文件

使用RPM或Debian软件包时，可以在系统配置文件中指定系统设置和环境变量，该文件位于：

| RPM | /etc/sysconfig/elasticsearch |
| ------ | ------ |
| Debian | /etc/default/elasticsearch |

但是，对于使用的systemd系统，需要通过systemd指定系统限制。

#### 系统配置

在使用systemd的系统上使用RPM或Debian软件包时 ，必须通过systemd指定系统限制。

systemd服务文件（/usr/lib/systemd/system/elasticsearch.service）包含默认应用的限制。

要覆盖它们，请添加一个名为的文件 /etc/systemd/system/elasticsearch.service.d/override.conf（或者，您可以运行sudo systemctl edit elasticsearch它在默认编辑器中自动打开文件）。设置此文件中的任何更改，例如：

```config
[Service]
LimitMEMLOCK=infinity
```
完成后，运行以下命令重新加载单位：

`sudo systemctl daemon-reload`

### 禁用swap

大多数操作系统尝试使用尽可能多的内存来存储文件系统缓存，并急切地交换掉未使用的应用程序内存。这可能导致部分JVM堆甚至其可执行页面被换出到磁盘。

交换对性能，节点稳定性非常不利，应该不惜一切代价避免。它可能导致垃圾收集持续数分钟而不是毫秒，并且可能导致节点响应缓慢甚至断开与群集的连接。在弹性分布式系统中，让操作系统终止节点更有效。

有三种禁用交换的方法。首选选项是完全禁用交换。如果这不是一个选项，是否更喜欢最小化swappiness与内存锁定取决于您的环境。

#### 禁用所有交换文件

通常Elasticsearch是在盒子上运行的唯一服务，其内存使用量由JVM选项控制。应该没有必要启用交换。

在Linux系统上，您可以通过运行以下命令暂时禁用交换：

`sudo swapoff -a`

这不需要重新启动Elasticsearch。

要永久禁用它，您需要编辑/etc/fstab文件并注释掉包含该单词的任何行swap。

在Windows上，可以通过完全禁用分页文件来实现等效System Properties → Advanced → Performance → Advanced → Virtual memory。

#### 配置swappiness

Linux系统上可用的另一个选项是确保将sysctl值 vm.swappiness设置为1。这降低了内核交换的倾向，在正常情况下不应导致交换，同时仍允许整个系统在紧急情况下交换。

#### 启用bootstrap.memory_lock

另一种选择是在Linux / Unix系统上使用mlockall，或 在Windows 上 使用 VirtualLock，以尝试将进程地址空间锁定到RAM中，从而防止任何Elasticsearch内存被换出。这可以通过将此行添加到config/elasticsearch.yml文件来完成：

`bootstrap.memory_lock: true`

> mlockall 如果尝试分配的内存超过可用内存，可能会导致JVM或shell会话退出！

启动Elasticsearch后，您可以通过检查mlockall此请求的输出中的值来查看是否已成功应用此设置：

`GET _nodes?filter_path=**.mlockall`

如果你看到mlockall的false，那么就意味着该mlockall 请求失败。您还会在日志中看到包含更多信息的行Unable to lock JVM Memory。

在Linux / Unix系统上，最可能的原因是运行Elasticsearch的用户没有锁定内存的权限。这可以像下面一样授权:

##### .zip 和 .tar.gz

在启动Elasticsearch之前 以root 用户身份运行 `ulimit -l unlimited`，或设置 `memlock` 为 `unlimitedin` `/etc/security/limits.conf`。

##### RPM和Debian

设置 `MAX_LOCKED_MEMORY` 成 `unlimited` 在 系统配置文件（或见下文使用系统 systemd）。

##### 系统使用 systemd

设置 `LimitMEMLOCK` 于 `infinity` 在systemd配置。

另一个 mlockall 可能失败的原因是 JNA 临时目录（通常是子目录/tmp）随 noexec 选项一起安装。这可以通过使用ES_JAVA_OPTS环境变量为JNA指定新的临时目录来解决：

```shell
export ES_JAVA_OPTS="$ES_JAVA_OPTS -Djna.tmpdir=<path>"
./bin/elasticsearch
```
或者在 `jvm.options` 配置文件中设置此JVM标志。

### 文件描述符

这仅适用于Linux和macOS，如果在Windows上运行Elasticsearch，则可以安全地忽略它。在Windows上，JVM使用 仅受可用资源限制的 API。

Elasticsearch使用大量文件描述符或文件句柄。用完文件描述符可能是灾难性的，最有可能导致数据丢失。确保将运行Elasticsearch的用户的打开文件描述符数量限制增加到65,536或更高。

对于.zipand .tar.gz packages，在启动Elasticsearch之前以root身份设置 `ulimit -n 65535`，或设置 nofile 为 65535 在 /etc/security/limits.conf 文件中。

在macOS上，您还必须将JVM选项传递-XX:-MaxFDLimit 给Elasticsearch，以便它使用更高的文件描述符限制。

RPM和Debian软件包已将文件描述符的最大数量默认为65535，无需进一步配置。

您可以max_file_descriptors使用Nodes Stats API 检查每个节点的配置，包括：

`GET _nodes/stats/process?filter_path=**.max_file_descriptors`

### 虚拟内存

Elasticsearch mmapfs默认使用目录来存储其索引。mmap计数的默认操作系统限制可能太低，这可能导致内存不足异常。

在Linux上，您可以通过以root用户运行以下命令来增加限制 :

`sysctl -w vm.max_map_count = 262144`

要永久设置此值，请在 /etc/sysctl.conf 中更新vm.max_map_count。要在重新启动后进行验证，请运行 sysctl vm.max_map_count。

RPM和Debian软件包将自动配置此设置。无需进一步配置

### 线程数

Elasticsearch为不同类型的操作使用许多线程池。重要的是它能够在需要时创建新线程。确保Elasticsearch用户可以创建的线程数至少为4096。

这可以通过ulimit -u 4096在启动Elasticsearch之前设置为root或通过设置nproc为4096in来完成 /etc/security/limits.conf。

作为服务运行时的程序包分发systemd将自动配置Elasticsearch进程的线程数。无需其他配置。

### DNS缓存设置

Elasticsearch 运行安全管理器。有了安全管理器，JVM默认无限期地缓存正主机名解析，默认为缓存负主机名解析10秒。

Elasticsearch 使用默认值覆盖此行为以将正向查找缓存 60 秒，并将负查找缓存 10 秒。

这些值应适用于大多数环境，包括 DNS 分辨率随时间变化的环境。

如果没有，你可以在 JVM 选项中编辑值 es.networkaddress.cache.ttl ，并 es.networkaddress.cache.negative.ttl 。

需要注意的是价值 networkaddress.cache.ttl=<timeout> 和 networkaddress.cache.negative.ttl=<timeout> 

在 Java安全策略由Elasticsearch忽略，除非你删除的设置 es.networkaddress.cache.ttl和es.networkaddress.cache.negative.ttl。

### JNA临时目录未安装 noexec 模式

> 仅适用linux系统

Elasticsearch使用Java Native Access（JNA）库来执行一些与平台相关的本机代码。

在Linux上，支持此库的本机代码在运行时从JNA存档中提取。

默认情况下，此代码将解压缩到Elasticsearch临时目录，该目录默认为子目录 /tmp。

或者，可以使用JVM标志控制此位置 -Djna.tmpdir=<path>。

作为本机库被映射到JVM虚拟地址空间为可执行，底层支架，该代码被提取到必须的位置的点不安装有noexec作为这防止能够这个代码映射为可执行JVM进程。

在某些强化的Linux安装中，这是一个默认的挂载选项/tmp。

安装底层安装的一个指示noexec是，在启动时，JNA将无法加载带有java.lang.UnsatisfiedLinkerError消息的异常failed to map segment from shared object。

请注意，异常消息可能因JVM版本而异。此外，依赖于通过JNA执行本机代码的Elasticsearch组件将失败并显示消息because JNA is not available。

如果您看到此类错误消息，则必须重新安装用于JNA的临时目录，以便不安装noexec。

## Bootstrap检查

总的来说，我们在遇到意外问题的用户方面有很多经验，因为他们没有配置 重要的设置。在以前的Elasticsearch版本中，其中一些设置的错误配置被记录为警告。可以理解，用户有时会错过这些日志消息。为了确保这些设置得到应有的关注，Elasticsearch在启动时进行了引导检查。

这些引导程序检查检查各种Elasticsearch和系统设置，并将它们与对Elasticsearch操作安全的值进行比较。如果Elasticsearch处于开发模式，则任何引导检查失败都会在Elasticsearch日志中显示为警告。如果Elasticsearch处于生产模式，则任何引导检查失败都会导致Elasticsearch拒绝启动。

有一些引导程序检查始终强制执行，以防止Elasticsearch使用不兼容的设置运行。这些检查单独记录。

### 开发与生产模式

默认情况下，Elasticsearch绑定到HTTP 和传输（内部）通信的环回地址。这适用于下载和使用Elasticsearch以及日常开发，但它对生产系统毫无用处。要加入群集，必须通过传输通信访问Elasticsearch节点。要通过非环回地址加入集群，节点必须将传输绑定到非环回地址，而不是使用单节点发现。因此，如果Elasticsearch节点无法通过非环回地址与另一台机器形成集群，则我们认为它处于开发模式，如果它可以通过非环回地址加入集群，则处于生产模式。

注意，HTTP和传输可以通过http.host和独立配置 transport.host; 这可以用于将单个节点配置为可通过HTTP访问以进行测试，而不会触发生产模式。

### 单节点发现

我们认识到一些用户需要将传输绑定到外部接口以测试其对传输客户端的使用。对于这种情况，我们提供发现类型single-node（通过设置discovery.type为 配置single-node）; 在这种情况下，节点将选择自己的主节点，并且不会与任何其他节点加入集群。

### 强制引导检查

如果您正在生产中运行单个节点，则可以避开引导检查（通过不绑定传输到外部接口，或通过绑定传输到外部接口并将发现类型设置为 single-node）。对于这种情况，您可以通过将系统属性设置es.enforce.bootstrap.checks为true （在设置JVM选项中设置此项，或通过添加-Des.enforce.bootstrap.checks=true 到环境变量中ES_JAVA_OPTS）来强制执行引导程序检查。如果您处于这种特定情况，我们强烈建议您这样做。此系统属性可用于强制执行独立于节点配置的引导程序检查。


### 堆大小检查

如果以不等的初始和最大堆大小启动JVM，则在系统使用期间调整JVM堆的大小时，它可能会暂停。

要避免这些调整大小暂停，最好以初始堆大小等于最大堆大小来启动JVM。

此外，如果 bootstrap.memory_lock启用，JVM将在启动时锁定堆的初始大小。

如果初始堆大小不等于最大堆大小，则在调整大小后不会出现所有JVM堆都锁定在内存中的情况。要传递堆大小检查，必须配置堆大小。

### 文件描述符检查

文件描述符是用于跟踪打开的“文件”的Unix构造。但在Unix中，一切都是文件。例如，“文件”可以是物理文件，虚拟文件​​（例如/proc/loadavg）或网络套接字。Elasticsearch需要大量文件描述符（例如，每个分片由多个段和其他文件组成，加上与其他节点的连接等）。此引导程序检查在OS X和Linux上强制执行。要传递文件描述符检查，您可能必须配置文件描述符。

### 内存锁定检查

当JVM执行主要的垃圾收集时，它会触及堆的每个页面。如果这些页面中的任何页面被换出到磁盘，则必须将它们交换回内存。这会导致很多磁盘颠簸，Elasticsearch更愿意使用它来处理请求。有几种方法可以配置系统以禁止交换。一种方法是请求JVM通过mlockall（Unix）或虚拟锁（Windows）将堆锁定在内存中。这是通过Elasticsearch设置完成的 bootstrap.memory_lock。但是，有时可以将此设置传递给Elasticsearch，但Elasticsearch无法锁定堆（例如，如果elasticsearch 用户没有memlock unlimited）。该内存锁定检查验证，如果该bootstrap.memory_lock设置已启用，JVM已成功锁定堆。要通过内存锁定检查，您可能需要进行配置bootstrap.memory_lock。

### 最大线程数检查

Elasticsearch通过将请求分解为多个阶段并将这些阶段交给不同的线程池执行程序来执行请求。Elasticsearch中的各种任务有不同的线程池执行程序。因此，Elasticsearch需要能够创建大量线程。最大线程数检查可确保Elasticsearch进程有权在正常使用情况下创建足够的线程。此检查仅在Linux上强制执行。如果您使用的是Linux，要传递最大线程数检查，则必须配置系统以允许Elasticsearch进程创建至少4096个线程。这可以通过/etc/security/limits.conf 使用nproc设置来完成（请注意，您可能还必须增加root用户的限制）。

### 最大文件大小检查

作为各个分片的组件的分段文件和作为translog组件的translog代可以变大（超过几千兆字节）。在可以由Elasticsearch进程创建的文件的最大大小有限的系统上，这可能导致写入失败。因此，这里最安全的选项是最大文件大小是无限的，这是最大文件大小引导程序检查强制执行的。要传递最大文件检查，必须配置系统以允许Elasticsearch进程编写无限大小的文件。这可以通过 /etc/security/limits.conf使用fsize设置来完成unlimited（请注意，您可能还必须增加root用户的限制）。

### 最大的虚拟内存检查

Elasticsearch和Lucene使用mmap很好的效果将索引的部分映射到Elasticsearch地址空间。这样可以将某些索引数据从JVM堆中移除，但在内存中可以实现快速访问。为了使其有效，Elasticsearch应具有无限的地址空间。最大大小的虚拟内存检查强制Elasticsearch进程具有无限的地址空间，并且仅在Linux上强制执行。要通过最大大小的虚拟内存检查，必须配置系统以允许Elasticsearch进程具有无限的地址空间。这可以通过/etc/security/limits.conf使用as设置来完成unlimited（请注意，您可能还必须增加root用户的限制）。

### 最大地图计数检查

继续前一点，为了mmap有效使用，Elasticsearch还需要能够创建许多内存映射区域。最大映射计数检查检查内核是否允许进程至少具有262,144个内存映射区域，并且仅在Linux上强制执行。要通过最大映射计数检查，必须至少配置vm.max_map_countvia 。sysctl262144

或者，仅当您使用mmapfs或hybridfs作为索引的商店类型时，才需要检查最大映射计数 。如果您不允许使用，mmap则不会强制执行此引导程序检查

### 客户端JVM检查

OpenJDK派生的JVM提供了两种不同的JVM：客户端JVM和服务器JVM。这些JVM使用不同的编译器从Java字节码生成可执行的机器代码。客户端JVM针对启动时间和内存占用进行了调整，同时调整服务器JVM以最大限度地提高性能。两个VM之间的性能差异可能很大。客户端JVM检查确保Elasticsearch未在客户端JVM中运行。要通过客户端JVM检查，必须使用服务器VM启动Elasticsearch。在现代系统和操作系统上，服务器VM是默认设置。

### 使用串行收集器检查

针对不同工作负载的OpenJDK派生JVM有各种垃圾收集器。特别是串行收集器最适合单个逻辑CPU机器或非常小的堆，它们都不适合运行Elasticsearch。在Elasticsearch中使用串行收集器可能会对性能造成破坏性影响。串行收集器检查可确保Elasticsearch未配置为与串行收集器一起运行。要通过串行收集器检查，您不能使用串行收集器启动Elasticsearch（无论它是来自您正在使用的JVM的默认值，还是您已明确指定它-XX:+UseSerialGC）。请注意，Elasticsearch附带的默认JVM配置会将Elasticsearch配置为使用CMS收集器。

### 系统调用过滤器检查

Elasticsearch根据操作系统安装各种风格的系统调用过滤器（例如，Linux上的seccomp）。安装这些系统调用过滤器是为了防止执行与分叉相关的系统调用的能力，作为防御Elasticsearch上任意代码执行攻击的防御机制。系统调用筛选器检查确保如果启用了系统调用筛选器，则表明已成功安装它们。要通过系统调用过滤器检查，您必须修复系统上阻止系统调用过滤器安装的任何配置错误（检查日志），或者通过设置为自行风险禁用系统调用过滤器。bootstrap.system_call_filterfalse

### OnError和OnOutOfMemoryError检查

JVM选项OnError，OnOutOfMemoryError如果JVM遇到致命错误（OnError）或 OutOfMemoryError（OnOutOfMemoryError），则启用执行任意命令。但是，默认情况下，启用Elasticsearch系统调用过滤器（seccomp），这些过滤器会阻止分叉。因此，使用OnError或OnOutOfMemoryError 和系统调用过滤器是不兼容的。该OnError和 OnOutOfMemoryError检查防止Elasticsearch从如果这两个JVM选项的使用和系统调用过滤器可启动。始终强制执行此检查。要通过这项检查没有启用 OnError，也没有OnOutOfMemoryError; 相反，升级到Java 8u92并使用JVM标志ExitOnOutOfMemoryError。虽然这并不具有的全部功能OnError，也没有OnOutOfMemoryError，任意分叉不会被启用支持的Seccomp。

### 早期检查

OpenJDK项目提供即将发布的早期访问快照。这些版本不适合生产。早期访问检查会检测这些早期访问快照。要通过此检查，您必须在JVM的发布版本上启动Elasticsearch。

### G1GC检查

已知JDK 8附带的早期版本的HotSpot JVM在启用G1GC收集器时会出现可能导致索引损坏的问题。受影响的版本早于JDK 8u40附带的HotSpot版本。G1GC检查检测到HotSpot JVM的这些早期版本。

### 所有权限检查

all权限检查可确保在引导期间使用的安全策略不会授予java.security.AllPermissionElasticsearch。使用所有权限运行等同于禁用安全管理器。

### 发现配置检查

默认情况下，当Elasticsearch首次启动时，它将尝试发现在同一主机上运行的其他节点。如果在几秒钟内未发现任何选定的主服务器，则Elasticsearch将形成一个包含已发现的任何其他节点的集群。在开发模式下无需任何额外配置即可形成此集群非常有用，但这不适合生产，因为它可能会形成多个集群并因此丢失数据。

此引导检查可确保未使用默认配置运行发现。可以通过设置以下至少一个属性来满足：

* discovery.seed_hosts
* discovery.seed_providers
* cluster.initial_master_nodes

## 启动Elasticsearch

启动Elasticsearch的方法因安装方式而异。

### 归档包（.tar.gz）

如果您使用.tar.gz软件包安装了Elasticsearch ，则可以从命令行启动Elasticsearch。

#### 从命令行编辑运行Elasticsearch

可以从命令行启动Elasticsearch，如下所示：

`./bin/elasticsearch`

默认情况下，Elasticsearch在前台运行，将其日志打印到标准输出（stdout），然后按下即可停止Ctrl-C。

> 与Elasticsearch一起打包的所有脚本都需要一个支持数组的Bash版本，并假设Bash可用于/bin/bash。因此，Bash应该直接或通过符号链接在此路径上可用。

#### 作为守护进程编辑运行

要将Elasticsearch作为守护程序运行，请-d在命令行中指定，并使用以下-p选项将进程ID记录在文件中：

`./bin/elasticsearch -d -p pid`

可以在$ES_HOME/logs/目录中找到日志消息。

要关闭Elasticsearch，请终止pid文件中记录的进程ID ：

`pkill -F pid`

> RPM和Debian 软件包中提供的启动脚本负责为您启动和停止Elasticsearch进程。

### 归档包（.zip）

如果您在Windows上使用.zip软件包安装Elasticsearch ，则可以从命令行启动Elasticsearch。如果您希望Elasticsearch在启动时自动启动而无需任何用户交互，请将Elasticsearch安装为服务。

#### 从命令行编辑运行Elasticsearch

可以从命令行启动Elasticsearch，如下所示：

`.\bin\elasticsearch.bat`

默认情况下，Elasticsearch在前台运行，将其日志打印到STDOUT，并可以通过按下来停止Ctrl-C。

### Debian软件包

安装后Elasticsearch不会自动启动。如何启动和停止Elasticsearch取决于您的系统是使用SysV init还是 systemd（由较新的发行版使用）。您可以通过运行此命令来判断正在使用哪个：

`ps -p 1`

#### 使用SysV 编辑运行Elasticsearchinit

使用此update-rc.d命令将Elasticsearch配置为在系统启动时自动启动：

`sudo update-rc.d elasticsearch defaults 95 10`

可以使用以下service命令启动和停止Elasticsearch ：

```shell
sudo -i service elasticsearch start
sudo -i service elasticsearch stop
```

如果Elasticsearch因任何原因无法启动，它将打印STDOUT失败的原因。可以在中找到日志文件/var/log/elasticsearch/。

#### 使用systemd方式运行elasticsearch

要将Elasticsearch配置为在系统启动时自动启动，请运行以下命令：

```shell
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```

Elasticsearch可以按如下方式启动和停止：

```shell
sudo systemctl start elasticsearch.service
sudo systemctl stop elasticsearch.service
```

这些命令不提供有关Elasticsearch是否成功启动的反馈。相反，此信息将写入位于的日志文件中/var/log/elasticsearch/。

默认情况下，Elasticsearch服务不会在systemd 日记中记录信息。要启用journalctl日志记录，--quiet必须从文件中的ExecStart命令行中删除该选项elasticsearch.service。

当systemd启用了日志记录，日志信息使用可用journalctl的命令：

附：

`sudo journalctl -f`

列出elasticsearch服务的日记帐分录：

`sudo journalctl --unit elasticsearch`

要从给定时间开始列出elasticsearch服务的日记帐分录：

`sudo journalctl --unit elasticsearch --since  "2016-10-30 18:17:16"`

有关更多命令行选项，请查看man journalctl或https://www.freedesktop.org/software/systemd/man/journalctl.html。

### Docker图像

如果安装了Docker镜像，则可以从命令行启动Elasticsearch。根据您使用的是开发模式还是生产模式，有不同的方法。请参阅从命令行运行Elasticsearch。

### MSI包

如果您使用该.msi软件包在Windows上安装了Elasticsearch ，则可以从命令行启动Elasticsearch。如果您希望它在启动时自动启动而无需任何用户交互， 请将Elasticsearch安装为Windows服务。

#### 从命令行编辑运行Elasticsearch

安装后，可以从命令行启动Elasticsearch，如果未作为服务安装并配置为在安装完成时启动，如下所示：

`.\bin\elasticsearch.exe`

命令行终端将显示类似于以下内容的输出：

默认情况下，Elasticsearch在前台运行，STDOUT除了<cluster name>.log内部文件之外还打印其日志LOGSDIRECTORY，可以通过按下来停止Ctrl-C。

### RPM包

安装后Elasticsearch不会自动启动。如何启动和停止Elasticsearch取决于您的系统是使用SysV init还是 systemd（由较新的发行版使用）。您可以通过运行此命令来判断正在使用哪个：

`ps -p 1- p 1`

#### 使用SysV 编辑运行Elasticsearchinit

使用此chkconfig命令将Elasticsearch配置为在系统启动时自动启动：

`sudo chkconfig --add elasticsearch`

可以使用以下service命令启动和停止Elasticsearch ：

```shell
sudo -i service elasticsearch start
sudo -i service elasticsearch stop
```

如果Elasticsearch因任何原因无法启动，它将打印STDOUT失败的原因。可以在中找到日志文件/var/log/elasticsearch/。

#### 使用编辑运行Elasticsearchsystemd

要将Elasticsearch配置为在系统启动时自动启动，请运行以下命令：

```shell
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```

Elasticsearch可以按如下方式启动和停止：

```shell
sudo systemctl start elasticsearch.service
sudo systemctl stop elasticsearch.service
```

这些命令不提供有关Elasticsearch是否成功启动的反馈。相反，此信息将写入位于的日志文件中/var/log/elasticsearch/。

默认情况下，Elasticsearch服务不会在systemd 日记中记录信息。要启用journalctl日志记录，--quiet必须从文件中的ExecStart命令行中删除该选项elasticsearch.service。

当systemd启用了日志记录，日志信息使用可用journalctl的命令：

尾随期刊：

`sudo journalctl -f`

列出elasticsearch服务的日记帐分录：

`sudo journalctl --unit elasticsearch`

要从给定时间开始列出elasticsearch服务的日记帐分录：

`sudo journalctl --unit elasticsearch --since  "2016-10-30 18:17:16"`

有关更多命令行选项，请查看man journalctl或https://www.freedesktop.org/software/systemd/man/journalctl.html。

## 停止Elasticsearch

有序关闭Elasticsearch可确保Elasticsearch有机会清理和关闭未完成的资源。例如，以有序方式关闭的节点将从群集中删除自身，将translog同步到磁盘，并执行其他相关的清理活动。您可以通过正确停止Elasticsearch来帮助确保有序关闭。

如果您将Elasticsearch作为服务运行，则可以通过安装提供的服务管理功能停止Elasticsearch。

如果您直接运行Elasticsearch，则可以通过在控制台中运行Elasticsearch时发送control-C或通过发送SIGTERM到POSIX系统上的Elasticsearch进程来停止Elasticsearch 。您可以通过各种工具（例如，ps或jps）获取PID以发送信号：

```shell
$ jps | grep Elasticsearch
14542 Elasticsearch
```

从Elasticsearch启动日志：

`[2016-07-07 12:26:18,908][INFO ][node                     ] [I8hydUG] version[5.0.0-alpha4], pid[15399], build[3f5b994/2016-06-27T16:23:46.861Z], OS[Mac OS X/10.11.5/x86_64], JVM[Oracle Corporation/Java HotSpot(TM) 64-Bit Server VM/1.8.0_92/25.92-b14]```

或者通过在启动时指定写入PID文件的位置（-p <path>）：

```shell
$ ./bin/elasticsearch -p /tmp/elasticsearch-pid -d
$ cat /tmp/elasticsearch-pid && echo
15516
$ kill -SIGTERM 15516
```

### 停止致命错误

在Elasticsearch虚拟机的生命周期中，可能会出现某些致命错误，导致虚拟机处于可疑状态。此类致命错误包括内存不足错误，虚拟机内部错误以及严重的I / O错误。

当Elasticsearch检测到虚拟机遇到此类致命错误时，Elasticsearch将尝试记录错误，然后暂停虚拟机。当Elasticsearch启动此类关闭时，它不会按上述顺序关闭。Elasticsearch进程还将返回一个特殊的状态代码，指示错误的性质。

| JVM内部错误      | 128  |
| ---------------- | ---- |
| 内存不足错误     | 127  |
| 堆栈溢出错误     | 126  |
| 未知的虚拟机错误 | 125  |
| 严重的I/O错误    | 124  |
| 未知的致命错误   | 1    |

## 将节点添加到群集

当您启动Elasticsearch的实例时，您正在启动一个节点。Elasticsearch 集群 是一组具有相同cluster.name属性的节点。当节点加入或离开集群时，集群会自动重组自身，以便在可用节点之间均匀分布数据。

如果您正在运行Elasticsearch的单个实例，则您拥有一个节点的集群。所有主分片都驻留在单个节点上。不能分配副本分片，因此群集状态保持黄色。群集功能齐全，但在发生故障时存在数据丢失的风险。

![elas_0202.png](./files/elas_0202.png)

您将节点添加到群集以提高其容量和可靠性。默认情况下，节点既是数据节点又有资格被选为控制集群的主节点。您还可以为特定目的配置新节点，例如处理摄取请求。有关更多信息，请参阅 节点。

向集群添加更多节点时，它会自动分配副本分片。当所有主分片和副本分片都处于活动状态时，群集状态将变为绿色。

![elas_0204.png](./files/elas_0204.png)

要将节点添加到群集：

1. 设置一个新的Elasticsearch实例。
1. 在其cluster.name属性中指定群集的名称。例如，将节点添加到logging-prod集群中，设置cluster.name: "logging-prod" 在elasticsearch.yml。
1. 启动Elasticsearch。该节点自动发现并加入指定的集群。

有关发现和分片分配的详细信息，请参阅 发现和群集构成以及分片分配和群集级路由。

## 设置X-Pack

收费功能, 算了 

## 配置监控 xpack

## 配置安全性 xpack

## 配置X-Pack Java客户端 xpack

## Bootstrap检查X-Pack xpack
