---
layout:     post
title:      Linux操作系统技术
subtitle:   dnf - module 使用
date:       2020-11-23
author:     李绍俊
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Linux
    - dnf
    - 包管理技术
---

# CentOS8 dnf module更换软件流

博主： 乾坤盘 发布时间：2020 年 02 月 04 日 851次浏览 暂无评论 3221字数 分类： Linux
首页正文 分享到：
CentOS 8的 dnf 新增了的一个moduler 功能，中文译意的意思是模块流(大致如下)，该功能主要用于切换不同版本的软件，其主要用于快速替换升级当前使用软件版本。

Fedora AppStream

如果用过Windows 下的phpMyStudy 应该会知道，里面有一个一键切换PHP版本的功能，CentOS 8 中的dnf module 也是用于实现类似功能的，例如切换php、nginx、nodejx等软件版本的，后续CentOS8还会推出更多module的(这些module大部分集中在 AppStream软件库中)。
同时已经有部分第三方软件库支持该功能了，例如，remi 这个第三方源(repo下载:CentOS8 yum/dnf 配置)

其实在平时使用的时候其实就已经使用了moduler功能了，只是有的时候会被忽略而已。

dnf install php

dnf Moduler 使用
基础使用方法

dnf [OPTIONS] module [COMMAND] [MODULE-SPEC]

OPTIONS:
详情查询 dnf(8) 的 man 帮助文档

COMMAND:
enable 启用模块
info 查询模块信息
remove 卸载模块
provides 查询模块的提供软件库信息
list 查询模块的详细信息
update 更新模块
install 安装模块
reset 重置模块
disable 禁用模块

MODULE-SPEC:
Name[:Stream[/Profiles]] 模块名称[:流[/配置]]
查询有哪些模块流
查询指定软件的模块流，输入命令dnf module list php就可以看到指定软件提供的所有模块流了。

dnf module list php

CentOS AppStream - 8表示一个软件库(repo)中包含的模块流，每一行代表一个模块流。
一共有4列，分别是 Name(名称)，Stream(模块流)，Profiles(配置)，Summary(简介)。
其中Profiles列中的[d]标志着，在未指定配置时，将默认将使用此配置。
而Stream列中[d]标志着，在未指定模块流时，将默认使用该软件流。

查询所有软件流
如果想要查询所有软件流，可以不输入软件名称，直接输入命令dnf module list，如下:

dnf module list

安装指定的模块流
如果想要安装指定模块流的软件可以直接使用命令dnf module install php:7.2，如下图：

dnf module install php:7.2

当然没如果只是想要启用指定模块流而不想要安装软件，可以使用此命令dnf module enable php:7.3/common。
注意，由上面的查询我们可以看见，php7.3没有指定任何默认选项，所以这里的 MODULE-SPEC 需要写全。

dnf module enable php:7.3/common

更换指定的模块流
dnf同时支持升级和降级两种更换模块流的方法，下面将演示这两种的使用方法：
升级模块流

首先需要重置模块流，注意不用卸载先前安装的软件！！

dnf module reset php

直接安装更高版本的PHP即可(这里使用的时remi软件库)，如果有冲突，那么dnf会自动将其升级未对应版本的模块流。
这样就完成了模块流的升级

dnf module install php:remi-7.4/common

降级模块流

与升级相同，先完成模块流的重置操作

dnf module reset php

安装低版本的模块流，同样的，dnf检测到软件冲突，自动完成软件的降级任务。

dnf module install php:7.2/common

更换模块流后的使用
在更换模块流之后，就可以安装平时使用的方法使用dnf安装软件了，可以不用管模块流之间冲突等问题了，这些都将由dnf自动完成处理。
例如在启用php:remi-7.4/common模块流之后(注意未安装)，可以直接安装php了，将会自动使用指定模块流的版本安装软件，如下图。

dnf module enable php:remi-7.4/common

----







# CentOS 8 安装第三方软件仓库（NodeJS/PHP 等）

2020年01月15日 1034点热度 1人点赞 2条评论

CentOS 8 正式发布有一段时间了，但是各大云服务商的镜像也就这段时间才准备好，比如阿里云前段时间才有 CentOS 8 的系统镜像。



作为一个爱尝鲜的人，[小山](https://github.com/Hill-98)第一时间把服务器迁移到了 CentOS 8，之前刚发布的时候只在虚拟机体验过，也没怎么折腾，但是真正到了生产环境，坑还是很多的。

CentOS 8 最大的坑莫过于安装软件包，CentOS 8 跟随上游 RHEL 8 引入了新的仓库 AppStream，这个仓库通过流式更新可以为 CentOS 带来新的版本，不会像以前那样，软件包过于陈旧。但是这样带来一个问题，如何保证用户的迁移成本，比如一些程序运行时，主板本对于大部分人来说不能轻易更新。解决问题的方法是引入模块化。模块化让软件包仓库可以同时分发相同软件的不同版本，一个模块代表一个版本，需要指定版本安装或启用对应模块即可，一个模块可以包含多个软件包以解决依赖性问题。

当你在 CentOS 8 添加新的没有模块的软件仓库，比如 NodeJS 的官方仓库，然后准备运行`dnf install nodejs`安装的时候，却发现软件版本没有变，并没有使用第三方软件仓库的包，这是因为模块拥有更高的优先级，而第三方软件仓库没有引入模块化或者是模块没有被启用，解决方法是禁用掉当前启用的模块或者启用新的模块。

![CenOS LOGO](https://upic-lisj.oss-cn-beijing.aliyuncs.com/uPic/1606108678-centos.png)

你可以通过`dnf module`命令来管理模块，常用的用法：

模块列表：`dnf module list`

安装模块：`dnf module install <module_spec>`

卸载模块：`dnf module uninstall <module_spec>`

启用模块：`dnf module enable <module_spec>`

禁用模块：`dnf module disable <module_spec>`

查看模块：`dnf module info <module_spec>`

关于 **module_spec** 取值，不同的命令有些许不同，先列出模块列表，可以看到四列输出，分别是：**Name、Stream、Profiles、Summary**，除了 **Summary** 之外其他三列都有用。

比如一个模块的 **Name** 是 nodejs，**Stream** 是 10，可以使用`dnf module install nodejs:10`来安装这个模块，也可以使用`dnf module disable nodejs:10`禁用掉这个模块，比如上面我们需要安装最新的 NodeJS 就需要禁用掉内置的模块，模块被禁用后，模块里的软件包就不会被安装，所以当前如果没有包含 nodejs 的另一个软件仓库，禁用以后，就无法使用 dnf 安装 nodejs。

**模块不需要启用也可以直接安装**

**如果有多个模块的 Name 相同，但 Stream 不同，在启用新的 Stream 之前必须禁用当前启用的 Stream。**



**Profiles** 列出的是当前模块可用的配置文件，如果没有指定文件配置文件，默认安装第一个。比如 PHP 模块有三个配置文件：`common, devel, minimal`，如果安装开发环境软件包，可以使用`dnf module install php:7.2/devel`这样的格式来指定配置文件，不过在启用/禁用/查看模块的命令不能使用这样的语法。
