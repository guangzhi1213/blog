---
title: "User"
date: 2021-04-25T11:25:17+08:00
lastmod: 2021-04-25T11:25:17+08:00
draft: false
keywords: []
description: ""
tags: ["linux","sre","ops"]
categories: ["linux"]
author: "王清"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

## 新建一个管理员用户

```shell
export JFUSER="wangqing"
export JFUSERPUB='ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDQPgrvrY/GZzUpL247IK7nYjVaeJS9PeuIgYEgo1O0uNYfKGte0BufsaBXJ/GUkGkuoEclv15Cz8ly8p+1tZ3TMWqZ53oNKfCvLl3/z4g7M3BhUzGLg8+vLsvXoshSp6Wo8GvhNr2p3z0rDuS9DZg7/q835ENYmy4nFoH3vTPIDCurzBYIOW6WwjYrYOzkc11SkCeXnSWB9RoTyTLlTL8lmrfuF05KrwKkpifabqnwxtOGA9VHPQ7/g+OzhkZYfQJ0dJLIq/LmsbgZMSQv6teXjjTN1P50CGWGr7HzDxgNoCkA4DuKcYPM6MPZBAarlcVElWXD7LhPfBQ2ndZaJSuP JF-005@Haier-PC'

adduser -g wheel ${JFUSER}

usermod -G wheel ${JFUSER}

passwd ${JFUSER}

echo -n "jfinfo88" | passwd --stdin ${JFUSER}

su - ${JFUSER} -c "mkdir /home/${JFUSER}/.ssh"

su - ${JFUSER} -c "chmod 700 /home/${JFUSER}/.ssh"

echo -n "${JFUSERPUB}" > /home/${JFUSER}/.ssh/authorized_keys

chown ${JFUSER}:wheel /home/${JFUSER}/.ssh/authorized_keys

#su - ${JFUSER} -c "echo -n "${JFUSERPUB}" > /home/${JFUSER}/.ssh/authorized_keys"

chmod 600 /home/${JFUSER}/.ssh/authorized_keys
```

### 私人公钥

```shell
wangwei_pubkey: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgQCeRXYKlctaWEOTTwPIbDoFWRhmi5/ytnxhazpMVf/o36rRMdyNoKsNZDRzmrT/J5X4fS28DG68pN+wRj/lnyQu1xzohWD8hgdYGkHhph4XjfIf+hZyrW5m+iDZ1HQGBQirtKVgpEpP1ivDMLxEsBjgbNtay7ggTCq2jpOTKXbYmw== wangwei@wangwei-PC
```

## 新建一个普通账号,如nginx用户

# Linux用户和组管理

from [闫秦川](https://www.jianshu.com/u/aaf2e477ba9f)

众所周知，Linux是一个多用户、多任务（Multi-Tasks、Multi-Users）的操作系统。那么Linux是如何区分和认证用户的，系统对每个用户的授权是如何管理的，出现问题如何追溯每个用户在系统内的操作记录，等等，这些就涉及到Linux中用户和组的管理。

> 1. AAA介绍
> 2. Linux用户类别
> 3. Linux用户标识（UID）
> 4. Linux组类别
> 5. Linux组标识（GID）
> 6. Linux中用户和组的相关数据库文件
> 7. Linux用户和组管理命令
>    (1) groupadd命令
>    (2) groupmod命令
>    (3) groupdel命令
>    (4) useradd命令
>    (5) usermod命令
>    (6) userdel命令
>    (7) passwd命令
>    (8) gpasswd命令
>    (9) newgrp命令
>    (10) chage命令
>    (11) id命令
>    (12) su命令
>    (13) 其它几个用户和组管理相关命令
> 8. Linux用户和组管理命令的相关示例
> 9. 通过更改用户和组的配置文件，直接添加或修改用户和组

首先介绍以下AAA。

### 1、AAA介绍

AAA指的是Authentication、Authorization、Accounting，即认证、授权和审计。

- 认证：验证用户是否可以获得权限，是3A的第一步，即验证身份；
- 授权：授权用户可以使用那些服务或资源，即身份验证成功后，赋予这个身份相应的权限；
- 审计：记录用户的操作情况，在Linux中，日志就是审计的一种手段。

Linux的用户和组管理可以说是基于AAA进行的，首先用户登录输入用户名密码，就是认证的过程；其次，在用户登录成功后，所拥有的权限各不相同，这就是授权；最后，用户的操作历史会记录在日志中，这是审计。
接下来介绍Linux中用户和组的类别，以及Linux是如何标识每个用户和组的：

### 2、Linux用户类别

Linux中，用户分为两大类、三小类：
分别为**管理员**（一般为root）和**普通用户** 。
普通用户中，又划分为两类，分别为*系统用户*和*登录用户*。

- 管理员
  即超级用户，可以操作系统中任意文件和命令，拥有最高的管理权限。
  Tips：一般情况下尽量不要使用root登录系统，避免误操作。

- 普通用户

  又分为登录用户和系统用户：

  - 登录用户
    一般为管理员手动添加的用户，默认仅拥有操作自身家目录中文件及目录的权限，以及进入与浏览相关目录文件的权限（如/etc、/var/log等），但没有创建、修改、删除等权限。
  - 系统用户
    一般为系统安装后默认存在的，且默认情况下不能登录系统，它们的存在主要是为了满足系统进程对文件属主的需求。
    Tips：在部署某些服务是，也可以手动添加某些系统用户。

### 3、Linux用户标识（UID）

Linux系统使用UID（User ID）来标识不同用户。
UID是16bits的二进制数字，所以换算成十进制，UID的范围是0~65535，Linux根据用户类别，对UID划分做了规定：

- 管理员
  UID为0
  Tips：当用户UID为0时，该用户就是管理员，所以不只root才是管理员，可以手动指定，但不建议。
- 普通用户（1~65535）
  - 系统用户
    一般发行版为1499（CentOS7为1999）
  - 登录用户
    一般发行版为50065535（CentOS7为100065535）

Tips：Linux是根据“名称解析库”（/etc/passwd）来进行用户名和UID的解析的，后面会详细介绍Linux中用户和组的相关信息库文件。

### 4、Linux组类别

Linux对组有三种划分方法：

- 第一种组类别，和用户划分类似，两大类三小类
  - 管理员组
  - 普通用户组（包括系统用户组和登录用户组）
- 第二种组类别
  - 用户的基本组（主组）
    用户必须有且只能有一个基本组。
  - 用户的附加组 （附属组）
    用户可以有0个、1个或多个附加组。
    基本组和附加组就比如，每个人有一个用来安家的房子（基本组），还可以有N个用于投资的房子（附属组）。
- 第三种组类别
  - 私有组
    每新建一个用户，如果不指定-g参数，都会自动创建一个和用户名同名的组，且组内只包含用户本身。
  - 公共组
    组内可包含多个用户。

### 5、Linux组标识（GID）

Linux系统使用GID（Group ID）来标识不同组。
GID的划分和UID相同，这里不再赘述。

### 6、Linux中用户和组的相关数据库文件

Linux中，与用户和组相关的信息主要存储在/etc/passwd、/etc/shadow、/etc/group三个文件中（存储格式中各字段用:分隔）：

- /etc/passwd：存储用户账户信息

  存储格式为，name:password:UID:GID:comment:directory:shell

  > - name：用户登录名；
  > - password：用户口令，用占位符x表示；
  > - UID：用户ID，用户登录时，系统根据UID，而非用户名来识别用户；
  > - GID：用户所属的主组ID；
  > - comment：用户的注释信息；
  > - directory：用户家目录的绝对路径；
  > - shell：用户的默认shell。

- /etc/shadow：存储用户密码信息

  存储格式为，

  登录名:$加密算法$salt$加密了的密码:最后一次更改密码的日期:密码最小期限:密码最大期限:密码警告时间段:密码禁用期:账户过期日期:保留字段

  > - 字段1：name用户登录名；
  > - 字段2：加密的密码，$为分隔符，首先是使用的加密算法，其次是salt（随机数），最后才是加密了的密码本身；
  > - 字段3：从1970年1月1日算起，密码被修改的天数（最近一次更改密码）；
  > - 字段4：密码最小期限，即密码最近更改日期到下次允许更改日期之间的天数（比如设置为10，则表示更改密码后10天内不允许再次更改；0表示无限制，可在任何时间修改）；
  > - 字段5：密码最大期限，密码最近更改日期到系统强制用户更改密码日期之间的天数（比如设置为100，则表示更改密码后100天，系统将强制要求再次更改密码；1表示永不修改）；
  > - 字段6：密码警告时间段，密码过期前，用户被警告的天数（比如，上个例子设置密码最大期限为100，密码警告时间段设为5，则表示更改密码后第96-100这5天，用户将被警告“密码即将过期”；-1表示没有警告）；
  > - 字段7：密码禁用期，密码过期后，到系统自动禁用账户的天数（-1表示永远不会禁用）；
  > - 字段8：账户过期日期（-1表示该账户被启用）；
  > - 字段9：保留条目，目前没用。

- /etc/group：存储用户组信息

  存储格式为，group_name:password:GID:user_list

  > - group_name：组名；
  > - password：用户组的口令，用占位符x表示，一般Linux用户组都没有口令；
  > - GID：组ID；
  > - user_list：用户列表，注意，这里列出的是以该组为附加组的用户列表，以此组为主组的用户没有列在此处。

### 7、Linux用户和组管理命令

> - 组管理：groupadd，groupmod，groupdel
> - 用户管理：useradd，usermod，userdel
> - 密码管理：passwd，gpasswd
> - 其它相关命令：newgrp，chage，chsh，id，su

#### (1) groupadd命令

- groupadd - create a new group
  新建组

- groupadd [options] group

  > - -g GID：指定GID：默认是上一个组的GID+1
  > - -r：创建系统组

- 例如，现在创建名为mygroup1和mygroup2两个组，查看其GID，分别为1000和1001，GID加1：
  `[root@localhost ~]# groupadd mygroup1;groupadd mygroup2`
  `[root@localhost ~]# tail -2 /etc/group`
  `mygroup1:x:1000:`
  `mygroup2:x:1001:`

- 再创建一个名为mygroup3的组，指定其GID为2222：
  `[root@localhost ~]# groupadd -g 2222 mygroup3`
  `[root@localhost ~]# tail -1 /etc/group`
  `mygroup3:x:2222:`

#### (2) gourpmod命令

- groupmod - modify a group definition on the system
  更改用户组属性

- groupmod [options] GROUP

  > - -g GID：--gid GID：修改GID
  > - -n NEW_NAME，修改组名

- 例如：将mygroup1的GID改为1111，组名改为MYGROUP：
  `[root@localhost ~]# groupmod -g 1111 -n MYGOURP mygroup1`
  `[root@localhost ~]# tail -1 /etc/group`
  `MYGOURP:x:1111:`

#### (3) groupdel命令

- groupdel - delete a group
  删除组
- groupdel [options] GROUP
- 当某user以某group为主组时，是无法使用groupdel命令删除该group的，但附加组不受影响
  `[root@localhost ~]# useradd user2 -g mygroup2 \\创建user2用户，指定主组为mygroup2`
  `[root@localhost ~]# groupdel mygroup2 \\删除mygroup2`
  `groupdel: cannot remove the primary group of user 'user2' \\提示无法删除`
  `[root@localhost ~]# useradd user3 -G mygroup3 \\新建user3用户，添加附加组mygroup3`
  `[root@localhost ~]# groupdel mygroup3 \\直接删除`

#### (4) useradd命令

- useradd - create a new user or update default new user information
  新建用户或修改新建用户时的默认属性

- useradd [options] LOGIN

  > - -u UID：--uid UID：指定UID，默认是上一个用户UID+1
  > - -g GROUP：--gid GROUP：指定用户的基本组，此组必须事先存在
  > - -G：--groups GROUP1,GROUP2...，指定用户的附加组，这些组必须事先存在
  > - -c COMMENT：--comment COMMENT：添加注释
  > - -d：--home HOME_DIR：指定用户家目录，通过复制/etc/skel并重命名实现的，指定的家目录路径如果事先存在，则不会为用户复制环境初始化配置文件（如.bashrc等）
  > - -s：--shell SHELL：指定用户默认shell，可用的所有shell列表存储在/etc/shells文件中
  > - -r：--system：创建系统用户

- 例如，创建suse用户，指定其UID为1100，指定其主组/基本组为slackware，指定附加组为group1，group2，添加注释“slackware management”，指定家目录为/home/susehome，默认shell为/bin/zsh：
  `[root@localhost ~]# useradd -u 1100 -g slackware -G "group1,group2" -c "slackware management" -d /home/susehome -s /bin/zsh suse`
  `[root@localhost ~]# tail -1 /etc/passwd`
  `suse:x:1100:2224:slackware management:/home/susehome:/bin/zsh`

- 这里-d指明的家目录事先不存在，所以会将骨架信息复制过来：
  `[root@localhost susehome]# ls -A /home/susehome/`
  `.bash_logout .bash_profile .bashrc .mozilla`

- 如果指定的家目录事先存在，则不会从/etc/skel复制信息：
  `[root@localhost home]# mkdir /home/suse2home`
  `[root@localhost home]# useradd -d /home/suse2home suse2`
  `useradd: warning: the home directory already exists.`
  `Not copying any file from skel directory into it.`
  `[root@localhost home]# ls -A /home/suse2home`
  `[root@localhost home]#`

- useradd -D [options]：显示或修改用户创建时的默认配置属性
  用户创建时的配置属性如下：

  > - GROUP：是否创建用户私有组，默认100，是
  > - HOME：家目录起始位置，默认/home
  > - INACTIVE：密码过期到用户注销的时间，默认-1，不注销
  > - EXPIRE：xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  > - SHELL：默认shell，默认为xxxxxxxxxxxxxxxxxxx
  > - SKEL：从哪复制用户骨架信息，默认/etc/skel
  > - CREATE_MAIL_SPOOL：是否创建邮件目录，默认yes，(在/var/spool/mail/)

- Tips：

  - 创建用户时的诸多默认配置保存在/etc/login.defs文件中；
  - useradd -D 修改的配置结果保存在/etc/default/useradd文件中。

- useradd -D显示的内容
  `[root@localhost home]# useradd -D`
  `GROUP=100`
  `HOME=/home`
  `INACTIVE=-1`
  `EXPIRE=`
  `SHELL=/bin/bash`
  `SKEL=/etc/skel`
  `CREATE_MAIL_SPOOL=yes`

- /etc/default/useradd文件中的内容：
  `[root@localhost mail]# cat /etc/default/useradd`
  `# useradd defaults file`
  `GROUP=100`
  `HOME=/home`
  `INACTIVE=-1`
  `EXPIRE=`
  `SHELL=/bin/bash`
  `SKEL=/etc/skel`
  `CREATE_MAIL_SPOOL=yes`

#### (5) usermod命令

- usermod - modify a user account
  修改用户属性

- usermod [options] LOGIN

  和useradd的选项大致相同

  > - -u UID：--uid UID：修改UID
  > - -g GROUP：--gid GROUP：修改用户的基本组，此组必须事先存在
  > - -G：--groups GROUP1,GROUP2...，修改用户的附加组，这些组须事先存在。
  >   注意，原来的附加组会被覆盖。
  >   如果只添加不覆盖，则配合使用-a选项。
  > - -a：--append：与-G一同使用，添加用户的附加组
  > - -c COMMENT：--comment COMMENT：修改注释
  > - -d：--home HOME_DIR：修改用户家目录
  >   用户原有的文件不会被转移至新位置。
  >   如果需要转移，则配合使用-m选项。
  > - -m：--move-home：只能与-d选项一同使用，用于将原来的家目录移动为新的家目录。
  > - -l：--login NEW_LOGIN：修改用户登录名
  > - -s：--shell SHELL：修改用户默认shell
  > - -L：--lock：锁定用户的密码，即禁止用户登录。
  >   其实就是在/etc/passwd文件中用户原来的密码字符串前添加一个“!”，使其不能匹配。
  > - -U：--unlock：解锁用户的密码

- usermod的常用选项和useradd相同，只需注意-d和-G两个选项

  - 例如只修改suse用户的家目录为/home/suse_newhome，不移动之前家目录的内容，则用-d选项（此目录须事先存在，否则只是更改了/etc/passwd中的记录，实际的目录是不会自动创建的）：
    `[root@localhost ~]# mkdir /home/suse_newhome`
    `[root@localhost ~]# usermod -d /home/suse_newhome suse`
    `[root@localhost ~]# ls -A /home/suse_newhome/`
    `[root@localhost ~]#`
  - 如果想修改家目录的同时，移动以前家目录的内容，则将-d和-m选项同时使用（这里要注意一下，目标目录不要事先存在，否则和只用-d的效果是一样的）：
    `[root@localhost ~]# rm -rf /home/suse_newhome/ \\这里先删除之前创建的目录`
    `[root@localhost ~]# usermod -md /home/suse_newhome suse`
    `[root@localhost ~]# cd ~suse`
    `[root@localhost suse_newhome]# ls -A`
    `.bash_logout .bash_profile .bashrc .mozilla`

#### (6) userdel命令

- userdel - delete a user account and related files
  删除用户账户和相关文件
- userdel [options] LOGIN
  - -r：删除用户时一并删除用户家目录
- userdel命令只需注意加不加-r选项的区别就可以，-r会在删除用户的同时，删除和用户相关的家目录和邮件文件。

#### (7) passwd命令

- passwd - update user's authentication tokens

- passwd [-k] [-l] [-u [-f]] [-d] [-e] [-n mindays] [-x maxdays] [-w warndays] [-i inactivedays] [-S] [--stdin] [username]

  > - passwd：不带任何选项：修改当前登录用户自己的密码
  > - passwd USER：修改指定用户的密码，默认仅root用户有此权限
  > - -l：--lock：锁定用户
  > - -u：--unlock：解锁用户
  > - -d：--delete：清除用户密码
  > - -e：--expire DATE：过期期限（日期）
  > - -i：--inactive DAYS：非活动期限（时长）
  > - -n：--minimum DAYS：密码的最短使用期限
  > - -m：--maximum DAYS：密码的最长使用期限
  > - -w：--warning DAYS：警告期限

- tips：

  - 修改密码也可以用如下命令：
    `echo "PASSWORD" | passwd --stdin USER \\多用于shell脚本中`

#### (8) gpasswd命令

- gpasswd - administer /etc/group and /etc/gshadow

- gpasswd [option] group

  - -a：--add USER：向组中添加用户
  - -d：--delete USER：从组中移除用户

- tips：

  - 组密码文件：/etc/gshadow

  - 组一般是没有密码的，给组设定密码的作用：避免用户随意切换基本组。

    > - newgrp GROUP：临时切换当前用户的基本组（exit：切换回之前的基本组）

#### (9) newgrp命令

- newgrp - log in to a new group
- newgrp [-] [group]
  - -：会模拟用户重新登录，以实现重新初始化其工作环境
  - exit：切换回去
- 例如，将root用户的主组临时切换为group1：
  `[root@localhost ~]# id -gn`
  `root`
  `[root@localhost ~]# newgrp - group1`
  `[root@localhost ~]# id -gn`
  `group1`

#### (10) chage命令

- chage - change user password expiry information
  修改密码的各类过期信息
- chage [options] LOGIN
  - -d，-E，-W，-m，-M
- chage命令用的不多，因为passwd命令中也可以修改密码的各类过期信息。

#### (11) id命令

- id - print real and effective user and group IDs（实际的和有效的ID是不同的）
- id [OPTION]... [USER]
  - id：不带任何选项：显示当前登录用户自己的信息
  - -u：--user：仅显示UID
  - -r：--real：仅显示实际的ID
  - -g：--group：仅显示用户的基本组ID
  - -G：--groups：仅显示用户所属的所有组的ID
  - -n：--name：显示名称，而非ID
- 例如，在suse用户下，使用id命令：
  `[suse@localhost ~]$ id`
  `uid=1100(suse) gid=2224(slackware) groups=2224(slackware),2225(group1),2226(group2) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023`
- 使用id -u，仅显示suse的UID：
  `[suse@localhost ~]$ id -u`
  `1100`
- 使用id -g，仅显示suse的主组GID，即slackware的GID：
  `[suse@localhost ~]$ id -g`
  `2224`
- 使用id -G，显示suse的主组和附加组的GID：
  `[suse@localhost ~]$ id -G`
  `2224 2225 2226`
- 配合-n选项，显示用户名或组名，而非ID号：
  `[suse@localhost ~]$ id -un`
  `suse`
  `[suse@localhost ~]$ id -gn`
  `slackware`
  `[suse@localhost ~]$ id -Gn`
  `slackware group1 group2`

#### (12) su命令

- su - run a command with substitute user and group ID
- su [options...] [-] [user [args...]]
  - 登录式切换：会通过重新读取目标用户的配置文件来重新初始化
    su - USER
    or
    su -l USER
  - 非登录式切换：不会读取目标用户的配置文件进行初始化
    su USER
  - -c COMMAND：仅以指定用户的身份运行此处指定的命令
- tips：
  管理员可无密码切换至其他任何用户

#### (13) 其它几个用户和组管理相关命令

- chsh：修改shell
- finger：查看信息
- chfn：修改finger信息
- whoami：我是谁呵呵
- pwck：检查用户信息是否有异常
- grpck：检查组信息是否有异常

### Linux用户和组管理命令的相关示例

- 创建组distro，其GID为2016；
  `[root@localhost ~]# groupadd -g 2016 distro`
  `[root@localhost ~]# tail -1 /etc/group`
  `distro:x:2016:`
- 创建用户mandriva, 其ID号为1005；基本组为distro；
  `[root@localhost ~]# useradd -u 1005 -g distro mandriva`
  `[root@localhost ~]# tail -1 /etc/passwd`
  `mandriva:x:1005:2016::/home/mandriva:/bin/bash`
- 创建用户mageia，其ID号为1100，家目录为/home/linux;
  `[root@localhost ~]# useradd -u 1100 -d /home/linux mageia`
  `[root@localhost ~]# tail -1 /etc/passwd`
  `mageia:x:1100:1100::/home/linux:/bin/bash`
- 给用户mageia添加密码，密码为mageedu；
  `[root@localhost ~]# echo "mageedu" | passwd --stdin mageia`
  `Changing password for user mageia.`
  `passwd: all authentication tokens updated successfully.`
- 删除mandriva，但保留其家目录；
  `[root@localhost ~]# userdel mandriva`
  `[root@localhost ~]# ls /home/ |grep linux`
  `linux`
- 创建用户slackware，其ID号为2002，基本组为distro，附加组peguin；
  `[root@localhost ~]# groupadd peguin`
  `[root@localhost ~]# useradd slackware -u 2002 -g distro -G peguin`
  `[root@localhost ~]# tail -1 /etc/passwd`
  `slackware:x:2002:2016::/home/slackware:/bin/bash`
- 修改slackware的默认shell为/bin/tcsh；
  `[root@localhost ~]# usermod -s /bin/tcsh slackware`
  `[root@localhost ~]# tail -1 /etc/passwd`
  `slackware:x:2002:2016::/home/slackware:/bin/tcsh`
- 为用户slackware新增附加组admins；
  `[root@localhost ~]# groupadd admins`
  `[root@localhost ~]# usermod -aG admins slackware`
  `[root@localhost ~]# id -Gn slackware`
  `distro peguin admins`

### 通过更改用户和组的配置文件，直接添加或修改用户和组

为了更深入了解用户和组的相关配置文件，可以手动更改配置文件以达到命令的执行效果。

1. 复制/etc/skel目录为/home/tuser1，要求/home/tuser1及其内部文件的属组和其它用户均没有任何访问权限。
   `[root@localhost ~]# cp -r /etc/skel /home/tuser1`
   \\这里我使用root用户复制，注意目标目录/home/tuser1不能事先存在，否则复制的结果为/home/tuser1/skel
   `[root@localhost ~]# ls -Al /home/tuser1/`
   `total 16`
   `-rw-r--r--. 1 root root 18 May 8 17:56 .bash_logout`
   `-rw-r--r--. 1 root root 193 May 8 17:56 .bash_profile`
   `-rw-r--r--. 1 root root 231 May 8 17:56 .bashrc`
   `drwxr-xr-x. 4 root root 4096 May 8 17:56 .mozilla`
   `[root@localhost ~]# chmod -R 700 /home/tuser1`
   `[root@localhost ~]# ll -al /home/tuser1`
   `total 24`
   `drwx------. 3 root root 4096 May 8 17:56 .`
   `drwxr-xr-x. 3 root root 4096 May 8 17:56 ..`
   `-rwx------. 1 root root 18 May 8 17:56 .bash_logout`
   `-rwx------. 1 root root 193 May 8 17:56 .bash_profile`
   `-rwx------. 1 root root 231 May 8 17:56 .bashrc`
   `drwx------. 4 root root 4096 May 8 17:56 .mozilla`
2. 编辑/etc/group文件，添加组hadoop。
   `[root@localhost ~]# vim + /etc/group`
   `...`
   `hadoop:x:1500:`
3. 手动编辑/etc/passwd文件新增一行，添加用户hadoop，其基本组ID为hadoop组的id号；其家目录为/home/hadoop。
   `[root@localhost ~]# vim + /etc/passwd`
   `...`
   `hadoop:x:1500:1500::/home/hadoop:/bin/bash`
   `[root@localhost ~]# id hadoop`
   `uid=1500(hadoop) gid=1500(hadoop) groups=1500(hadoop)`
4. 复制/etc/skel目录为/home/hadoop，要求修改hadoop目录的属组和其它用户没有任何访问权限。
   `[root@localhost ~]# cp -R /etc/skel /home/hadoop`
   `[root@localhost ~]# chmod 700 /home/hadoop`
   `[root@localhost ~]# chmod g=,o= /home/hadoop/`
   `[root@localhost ~]# ll -d /home/hadoop`
   `drwx------. 3 root root 4096 May 14 11:20 /home/hadoop`
5. 修改/home/hadoop目录及其内部所有文件的属主为hadoop，属组为hadoop。
   `[root@localhost ~]# chown -R hadoop:hadoop /home/hadoop/`
   `[root@localhost ~]# ll -a /home/hadoop/`
   `total 24`
   `drwx------. 3 hadoop hadoop 4096 May 14 11:20 .`
   `drwxr-xr-x. 4 root root 4096 May 14 11:20 ..`
   `-rw-r--r--. 1 hadoop hadoop 18 May 14 11:20 .bash_logout`
   `-rw-r--r--. 1 hadoop hadoop 193 May 14 11:20 .bash_profile`
   `-rw-r--r--. 1 hadoop hadoop 231 May 14 11:20 .bashrc`
   `drwxr-xr-x. 4 hadoop hadoop 4096 May 14 11:20 .mozilla`
   