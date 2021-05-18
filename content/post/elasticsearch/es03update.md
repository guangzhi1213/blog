---
title: "Es03update"
date: 2021-04-21T18:08:35+08:00
lastmod: 2021-04-21T18:08:35+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

# 升级 elasticsearch

Elasticsearch通常可以使用Rolling升级 过程进行升级，因此升级不会中断服务。支持滚动升级：

* 在次要版本之间
* 从5.6到6.7
* 从6.7到7.0.0

Elasticsearch可以读取在先前主要版本中创建的索引。如果您在5.x或之前创建了索引，则必须在升级到7.0.0之前重新索引或删除它们。如果存在不兼容的索引，Elasticsearch节点将无法启动。即使它们是由6.x群集创建的，5.x或更早索引的快照也无法还原到7.x群集。有关升级旧索引的信息，请参阅Reindex进行升级。

升级到新版本的Elasticsearch时，需要升级Elastic Stack中的每个产品。有关更多信息，请参阅“ 弹性堆栈安装和升级指南”。

要从6.6或更早版本直接升级到7.0.0，必须关闭群集，安装7.0.0并重新启动。欲了解更多信息，请参阅 完整集群重启升级。

## 准备升级

在升级Elasticsearch之前：

1. 检查弃用日志以查看您是否使用任何已弃用的功能并相应地更新代码。默认情况下，日志级别设置为时会记录弃用警告WARN。
1. 查看重大更改，并对7.0.0的代码和配置进行必要的更改。
1. 如果您使用自定义插件，请确保兼容版本可用。
1. 在升级生产群集之前，在开发环境中测试升级。
1. [备份您的数据](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-snapshots.html)！您必须拥有数据快照才能回滚到早期版本。

## 滚动升级

滚动升级允许Elasticsearch集群一次升级一个节点，因此升级不会中断服务。不支持在升级期间在同一群集中运行多个版本的Elasticsearch，因为无法将已升级的节点复制到运行旧版本的节点。

支持滚动升级：

* 在次要版本之间
* 从5.6到6.7
* 从6.7到7.0.0

从6.6或更早版本直接升级到7.0.0需要 [重新启动完整群集](https://www.elastic.co/guide/en/elasticsearch/reference/current/restart-upgrade.html)

要执行从6.7到7.0.0的滚动升级：

1. 禁用分片分配。
    关闭节点时，分配进程会等待 index.unassigned.node_left.delayed_timeout（默认情况下为1分钟），然后再开始将该节点上的分片复制到群集中的其他节点，这可能涉及大量I / O. 由于节点很快将重新启动，因此不需要此I / O. 您可以通过在关闭节点之前禁用副本分配来避免时间赛跑 ：
    ```es
    PUT _cluster/settings
    {
      "persistent": {
        "cluster.routing.allocation.enable": "primaries"
      }
    }
    COPY AS CURLVIEW IN CONSOLE 
    ```
1. 停止非必要索引并执行同步刷新。（可选的）

    虽然您可以在升级期间继续编制索引，但如果暂时停止非必要索引并执行同步刷新，则碎片恢复会快得多 。
    
    `POST _flush/synced`
    
    执行同步刷新时，请检查响应以确保没有失败。虽然请求本身仍返回200 OK状态，但响应正文中列出了由于挂起的索引操作而失败的同步刷新操作。如果有失败，请重新发出请求。
1. 暂时停止与活动机器学习作业和数据馈送相关的任务。（可选的）

    如果您的机器学习索引是在6.x之前创建的，则必须 [重新索引索引](https://www.elastic.co/guide/en/elasticsearch/reference/current/reindex-upgrade.html)。

    如果您的机器学习索引是在6.x中创建的，您可以：

    * 在升级期间保持机器学习作业运行。关闭机器学习节点时，其作业会自动移动到另一个节点并恢复模型状态。此选项使您的作业在升级期间继续运行，但会增加群集的负载。
    * 暂时停止与计算机学习作业和数据馈送相关的任务，并使用设置的升级模式API阻止打开新作业 ：

    * `POST _ml/set_upgrade_mode?enabled=true`

    +禁用升级模式时，作业将使用自动保存的最后一个模型状态恢复。此选项可避免在升级期间管理活动作业的开销，并且比明确停止数据馈送和关闭作业更快。

    * 停止所有数据馈送并关闭所有作业。此选项在关闭时保存模型状态。在升级后重新打开作业时，它们使用完全相同的模型。但是，保存最新模型状态比使用升级模式需要更长时间，尤其是当您有大量工作或具有大型模型状态的作业时。
1. 关闭单个节点。

    * 如果您正在运行Elasticsearch systemd：

        `sudo systemctl stop elasticsearch.service`
    * 如果您使用SysV运行Elasticsearch init：

        `sudo -i service elasticsearch stop`
    * 如果您将Elasticsearch作为守护程序运行：

        `kill $(cat pid)`
1. 升级您关闭的节点。

    要使用Debian或RPM软件包升级：
    * 使用rpm或dpkg安装新包。所有文件都安装在操作系统的适当位置，并且不会覆盖Elasticsearch配置文件。

    要使用zip或压缩tarball进行升级：

    1. 将zip或tarball解压缩到新目录。如果您不使用外部config和data目录，这一点至关重要。
    1. 设置ES_PATH_CONF环境变量以指定外部config目录和jvm.options文件的位置。如果未使用外部config目录，请将旧配置复制到新安装。
    1. 设置path.data在config/elasticsearch.yml指向您的外部数据目录。如果未使用外部data目录，请将旧数据目录复制到新安装。
    
    1. 重要
    1. 如果使用监视功能，请在升级Elasticsearch时重新使用数据目录。Monitoring通过使用存储在数据目录中的持久性UUID来标识唯一的Elasticsearch节点。
    
    1. 设置path.logs在config/elasticsearch.yml指向要保存你的日志的位置。如果未指定此设置，则日志将存储在您将存档解压缩到的目录中。

    > 当你解压ZIP压缩包或套餐，elasticsearch-n.n.n 目录包含Elasticsearch config，data，logs和 plugins目录。

    > 我们建议将这些目录移出Elasticsearch目录，以便在升级Elasticsearch时无法删除它们。要指定新位置，请使用ES_PATH_CONF环境变量和path.data和path.logs设置。有关更多信息，请参阅重要的Elasticsearch配置。

    > 在Debian的和RPM包放在这些目录中的每个操作系统的适当位置。在生产中，我们建议使用deb或rpm软件包进行安装。
1. 升级任何插件。

    使用该elasticsearch-plugin脚本安装每个已安装的Elasticsearch插件的升级版本。升级节点时，必须升级所有插件。
1. 如果使用Elasticsearch安全功能来定义域，请验证您的域设置是否是最新的。领域设置的格式在7.0版中已更改，特别是领域类型的位置已更改。请参阅 领域设置。
1. 启动已升级的节点。

    启动新升级的节点，并通过检查日志文件或提交_cat/nodes请求来确认它是否加入群集：

    ```GET _cat/nodes```
1. 重新启用分片。

    节点加入群集后，删除该cluster.routing.allocation.enable 设置以启用分片分配并开始使用该节点：

    ```es
    PUT _cluster/settings
    {
      "persistent": {
        "cluster.routing.allocation.enable": null
      }
    }
    ```
1. 等待节点恢复。

    在升级下一个节点之前，请等待群集完成分片分配。您可以通过提交_cat/health请求来检查进度：

    `GET _cat/health?v`

   等待status列切换yellow到green。一旦节点出现green，就分配了所有主分片和副本分片。

    > 在滚动升级期间，分配给运行新版本的节点的主分片无法将其副本分配给具有旧版本的节点。新版本可能具有旧版本无法理解的不同数据格式。

    > 如果无法将副本分片分配给另一个节点（群集中只有一个已升级的节点），则副本分片将保持未分配状态并保持状态yellow。

    > 在这种情况下，您可以在没有初始化或重新定位分片时继续（检查init和relo列）。

    > 一旦升级了另一个节点，就可以分配副本并且状态将更改为green。

    未同步刷新的碎片可能需要更长时间才能恢复。您可以通过提交_cat/recovery请求来监控各个分片的恢复状态：

    `GET _cat/recovery`
1. 重复

    当节点已恢复且群集稳定后，请对需要更新的每个节点重复这些步骤。

1. 重启机器学习工作。

    如果您暂时停止与计算机学习作业关联的任务，请使用set upgrade mode API将它们恢复为活动状态：

    `POST _ml/set_upgrade_mode?enabled=false`

    如果您在升级之前关闭了所有的机器学习作业，请打开作业，并开始从Kibana或与该数据传送专线空缺职位和 启动数据传送专线的API。

在滚动升级期间，群集继续正常运行。但是，任何新功能都将被禁用或以向后兼容模式运行，直到群集中的所有节点都升级为止。升级完成且所有节点都运行新版本后，新功能即可运行。一旦发生这种情况，就无法返回以向后兼容模式运行。运行先前主要版本的节点将不被允许加入完全更新的群集。

如果在升级过程中网络出现故障，将所有剩余的旧节点与群集隔离，则必须使旧节点脱机并升级它们以使其能够加入群集。

同样，如果运行仅包含一个主节点的测试/开发环境，则应最后升级主节点。重新启动单个主节点会强制重组群集。新群集最初将只具有升级的主节点，因此在重新加入群集时将拒绝旧节点。已升级的节点将成功重新加入已升级的主节点。

## 重启整个集群

1. 禁用分片分配。
    关闭节点时，分配进程会等待 index.unassigned.node_left.delayed_timeout（默认情况下为1分钟），然后再开始将该节点上的分片复制到群集中的其他节点，这可能涉及大量I / O. 由于节点很快将重新启动，因此不需要此I / O. 您可以通过在关闭节点之前禁用副本分配来避免时间赛跑 ：
    ```es
    PUT _cluster/settings
    {
      "persistent": {
        "cluster.routing.allocation.enable": "primaries"
      }
    }
    ```
1. 停止索引并执行同步刷新。

    执行同步刷新可加快碎片恢复速度。

    `POST _flush/synce`

    执行同步刷新时，请检查响应以确保没有失败。虽然请求本身仍返回200 OK状态，但响应正文中列出了由于挂起的索引操作而失败的同步刷新操作。如果有失败，请重新发出请求。
1. 暂时停止与活动机器学习作业和数据馈送相关的任务。（可选的）

    如果您的机器学习索引是在6.x之前创建的，则必须 [重新索引索引](https://www.elastic.co/guide/en/elasticsearch/reference/current/reindex-upgrade.html)。

    如果您的机器学习索引是在6.x中创建的，您可以：

    * 在升级期间保持机器学习作业运行。关闭机器学习节点时，其作业会自动移动到另一个节点并恢复模型状态。此选项使您的作业在升级期间继续运行，但会增加群集的负载。
    * 暂时停止与计算机学习作业和数据馈送相关的任务，并使用设置的升级模式API阻止打开新作业 ：

    * `POST _ml/set_upgrade_mode?enabled=true`

    +禁用升级模式时，作业将使用自动保存的最后一个模型状态恢复。此选项可避免在升级期间管理活动作业的开销，并且比明确停止数据馈送和关闭作业更快。

    * 停止所有数据馈送并关闭所有作业。此选项在关闭时保存模型状态。在升级后重新打开作业时，它们使用完全相同的模型。但是，保存最新模型
1. 关闭所有节点
    * systemd

    `sudo systemctl stop elasticsearch.service`

    * sysv,init

    `sudo -i service elasticsearch stop`

    * daemon

    `kill $(cat pid)`
1. 升级所有节点

    要使用Debian或RPM软件包升级：
    * 使用rpm或dpkg安装新包。所有文件都安装在操作系统的适当位置，并且不会覆盖Elasticsearch配置文件。

    要使用zip或压缩tarball进行升级：

    1. 将zip或tarball解压缩到新目录。如果您不使用外部config和data目录，这一点至关重要。
    1. 设置ES_PATH_CONF环境变量以指定外部config目录和jvm.options文件的位置。如果未使用外部config目录，请将旧配置复制到新安装。
    1. 设置path.data在config/elasticsearch.yml指向您的外部数据目录。如果未使用外部data目录，请将旧数据目录复制到新安装。
    
    1. 重要
    1. 如果使用监视功能，请在升级Elasticsearch时重新使用数据目录。Monitoring通过使用存储在数据目录中的持久性UUID来标识唯一的Elasticsearch节点。
    
    1. 设置path.logs在config/elasticsearch.yml指向要保存你的日志的位置。如果未指定此设置，则日志将存储在您将存档解压缩到的目录中。

    > 当你解压ZIP压缩包或套餐，elasticsearch-n.n.n 目录包含Elasticsearch config，data，logs和 plugins目录。

    > 我们建议将这些目录移出Elasticsearch目录，以便在升级Elasticsearch时无法删除它们。要指定新位置，请使用ES_PATH_CONF环境变量和path.data和path.logs设置。有关更多信息，请参阅重要的Elasticsearch配置。

    > 在Debian的和RPM包放在这些目录中的每个操作系统的适当位置。在生产中，我们建议使用deb或rpm软件包进行安装。
1. 升级任何插件。

    使用该elasticsearch-plugin脚本安装每个已安装的Elasticsearch插件的升级版本。升级节点时，必须升级所有插件。
1. 如果使用Elasticsearch安全功能来定义域，请验证您的域设置是否是最新的。领域设置的格式在7.0版中已更改，特别是领域类型的位置已更改。请参阅 领域设置。
1. 启动每个节点。

    如果您有专用主节点，请先启动它们并等待它们形成群集并在继续处理数据节点之前选择主节点。您可以通过查看日志来检查进度。

    如果从6.x群集升级，则必须通过设置设置来 配置群集引导cluster.initial_master_nodes。

    只要有足够的符合主节点的节点相互发现，它们就会形成一个集群并选出一个主节点。此时，您可以使用 _cat/health和_cat/nodes监视加入群集的节点：

    ```es
    GET _cat/health
    GET _cat/nodes
    ```
1. 等待所有节点加入群集并报告黄色状态。

    当节点加入群集时，它开始恢复本地存储的所有主分片。该_cat/healthAPI最初将报告status中red，表明并非所有的初级碎片已被分配。

    一旦节点恢复其本地分片，群集status将切换到yellow，表示已恢复所有主分片，但并未分配所有副本分片。这是预料之中的，因为您还没有重新启用分配。延迟副本的分配直到所有节点都yellow允许主服务器将副本分配给已经具有本地分片副本的节点。
1. 重新启用分配。

    当所有节点都已加入群集并恢复其主分片时，请通过恢复cluster.routing.allocation.enable其默认值来重新启用分配：

    ```es
    PUT _cluster/settings
    {
      "persistent": {
        "cluster.routing.allocation.enable": null
      }
    }
    ```

    重新启用分配后，群集开始将副本分片分配给数据节点。此时，恢复索引和搜索是安全的，但如果您可以等到所有主分区和副本分片都已成功分配并且所有节点的状态为，则群集将更快地恢复green。

    您可以使用_cat/health和 _cat/recoveryAPI 监视进度：

    ```es
    GET _cat/health
    GET _cat/nodes
    ```
1. 重启机器学习工作。

    如果您暂时停止与计算机学习作业关联的任务，请使用set upgrade mode API将它们恢复为活动状态：

    `POST _ml/set_upgrade_mode?enabled=false`

## 升级前重新创建索引

Elasticsearch可以读取在先前主要版本中创建的索引。如果您在5.x或之前创建了索引，则必须在升级到7.0.0之前重新索引或删除它们。如果存在不兼容的索引，Elasticsearch节点将无法启动。即使它们是由6.x群集创建的，5.x或更早索引的快照也无法还原到7.x群集。

此限制也适用于Kibana和X-Pack功能使用的内部索引。因此，在7.0.0中使用Kibana和X-Pack功能之前，必须确保内部索引具有兼容的索引结构。

您有两种方法可以重新索引旧索引：

* 在升级之前，在6.x群集上重新编制索引。
* 从远程 创建一个新的7.0.0群集和Reindex。这使您可以重新索引驻留在运行任何版本的Elasticsearch的集群上的索引。

> 升级基于时间的指数

> 如果使用基于时间的索引，则可能不需要将6.x之前的索引转发到7.0.0。随着时间的推移，基于时间的索引中的数据通常变得不那么有用，并且随着它们超过保留期而被删除。

> 除非您的保留期非常长，否则您可以等到升级到6.x，直到删除所有6.x之前的索引。

### reindex in place

您可以使用Kibana 6.7中的升级助手自动重新索引您需要转换为7.0.0的5.x索引。

要手动重新索引旧索引：

1. 使用7.x兼容映射创建索引。
1. 设置refresh_intervalto -1和number_of_replicasto以0进行有效的重建索引。
1. 使用reindexAPI将5.x索引中的文档复制到新索引中。您可以使用脚本在重建索引期间对文档数据和元数据执行任何必要的修改。
1. 重置旧索引中使用的值refresh_interval和number_of_replicas。
1. 等待索引状态更改为green。
1. 在单个更新别名请求中：

    * 删除旧索引。
    * 将具有旧索引名称的别名添加到新索引。
    * 将旧索引上存在的任何别名添加到新索引。

如果使用机器学习功能并且机器学习索引是在6.x之前创建的，则必须暂时停止与机器学习作业和数据馈送相关的任务，并防止在重新索引期间打开新作业。使用set upgrade mode API或 停止所有数据馈送并关闭所有计算机学习作业。

如果您使用Elasticsearch安全功能，则在重新索引.security*内部索引之前，最好在file 领域中创建临时超级用户帐户。

1. 在单个节点上，向域中添加临时超级用户帐户file。例如，运行elasticsearch-users useradd命令：

    `bin/elasticsearch-users useradd <user_name> -p <password> -r superuser`
1. 重新索引.security*索引时使用这些凭据。也就是说，使用它们登录Kibana并运行升级助手或调用reindex API。您可以使用常规管理凭据重新索引其他内部索引。
1. 从文件领域中删除临时超级用户帐户。例如，运行elasticsearch-users userdel命令：

    `bin/elasticsearch-users userdel <user_name>`

### 在远程集群冲洗创建索引

您可以使用来自远程的reindex将索引从旧群集迁移到新的7.0.0群集。这使您可以从6.7之前的群集迁移到7.0.0而不会中断服务。

> Elasticsearch提供向后兼容性支持，支持将先前主要版本的索引升级到当前主要版本。跳过主要版本意味着您必须自己解决任何向后兼容性问题。

> 如果使用机器学习功能并且从6.5或更早版本的群集迁移索引，则作业和数据源配置信息不会存储在索引中。您必须在新群集中重新创建机器学习作业。如果要从6.6或更高版本的群集进行迁移，最好暂时停止与机器学习作业和数据馈送相关的任务，以防止在稍微不同的时间重新编制索引的不同机器学习索引之间的不一致。使用 set upgrade mode API或 停止所有数据馈送并关闭所有计算机学习作业。

要迁移索引：

1. 设置新的7.0.0集群和现有群集添加到 reindex.remote.whitelist在elasticsearch.yml。

    `reindex.remote.whitelist: oldhost:9200`

    > 新群集不必开始完全扩展。在迁移索引并将负载转移到新群集时，可以将节点添加到新群集并从旧群集中删除节点。
1. 对于需要迁移到新集群的每个索引：

    1. 创建索引适当的映射和设置。设置 refresh_interval 为 -1和设置number_of_replicas 为 0,  更快的重建索引。
    1. 使用reindexAPI将远程索引中的文档提取到新的7.0.0索引中：

        ```es
        POST _reindex
        {
          "source": {
            "remote": {
              "host": "http://oldhost:9200",
              "username": "user",
              "password": "pass"
            },
            "index": "source",
            "query": {
              "match": {
                "test": "data"
              }
            }
          },
          "dest": {
            "index": "dest"
          }
        }
        ```

        如果通过设置wait_for_completion 为后台在后台运行reindex作业false，则reindex请求将返回一个task_id可用于监视任务API的reindex作业进度的程序： GET _tasks/TASK_ID。
    1. 重新索引作业完成后，将refresh_interval和 number_of_replicas设置为所需的值（默认设置为 30s和1）。
    1. 完成重建索引并且新索引的状态为green，您可以删除旧索引。


