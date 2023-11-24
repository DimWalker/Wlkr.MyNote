[![DimTechStudio.Com](vx_images/DimTechStudio-Logo.png)](https://www.dimtechstudio.com/)  

# [GitOps] 白嫖神器Github Actions，构建、推送Docker镜像一路畅通无阻

## 引言
当你没找到合适的基础的Docker镜像时，是否会一时冲动，想去自己构建。然而因为网络问题，各种软件官方源、镜像源（包括但不限于apt、pip、maven，npm），不但编译奇慢无比，还时常出错白白浪费几十个G的网络流量；硬件问题，爱用Windows但装WSL开了Hyper-V玩不了模拟器，用虚拟机又觉得麻烦占磁盘空间，还没钱续期云服务器……
我的朋友，如果你还在为以上种种问题而苦恼，那么我很荣幸为你推荐<span style='color:red;'>~~Nuget~~</span> GitOps，Github Actions!  
GitOps，作为一种现代化的运维理念，强调通过版本控制系统来管理基础设施和应用配置，提高可维护性和可靠性。在这个背景下，Github Actions作为一项强大的CI/CD工具，为我们提供了优雅的解决方案。本文将深入探讨如何充分利用Github Actions服务，白嫖其强大的构建能力，实现Docker镜像的自动构建与推送，确保网络畅通无阻。  
【不得不说同样是官方源，Nuget就极少像前面几个源出问题，.net yyds！】

## Github Actions简介
Github Actions是Github提供的一项持续集成和持续交付服务，与仓库无缝集成，可通过简单的YAML配置文件定义工作流程。这使得我们能够轻松地在代码仓库中管理和执行CI/CD任务，提高开发和部署的效率。借助Github Actions，我们可以构建、测试和部署项目，将整个开发周期变得更加流畅。  
GitHub可以提供 Linux、Windows 和 macOS 虚拟机来运行你的工作流程，在使用Github Actions之前，你需要了解以下前置知识:  
* Yaml基础语法
* Linux（或Windows或macOS）脚本相关知识

### 什么是Yaml
当创建Github Actions时，会在代码库.github/workflows目录下，创建一个.yml 文件，每个yml对应一个工作流。
YAML（YAML Ain't Markup Language或YAML是一个人类可读的数据序列化格式）是一种简洁且易读的数据标记语言。它常被用于配置文件和数据交换格式，以人类可读的方式表示数据结构，有以下几个特点:
* 大小写敏感。
* 使用缩进表示层级关系。
* 缩进只能使用空格，不能用 TAB 字符。
* 缩进的空格数量不重要，只要层级相同的元素左对齐即可。
* # 表示注释。

### Github Actions的Yaml结构
* name：workflow名称
* on：触发器
* env：自定义的环境变量
* job：一个workflow可以有多个job，每个job包含多个step
* runs-on：任务容器，如ubuntu-latest，windows-latest，macos-latest
* step：任务步骤
* action：每个步骤可以执行多个命令

### Github Actions的使用限制
Github Actions可以免费使用，也可以付费使用，其中免费用户有以下限制
* 使用时长
可以每月使用2000分钟，存储500MB（应该是指仓库大小），其中不同容器时间系数是不同的，Linux是1，Windows是2（1000分钟），MacOS是10（200分钟）；
* 并发作业数20
* 作业执行时间最长6小时
* 工作流运行时间每次工作流运行时间限制为 35 天
（笔者没看懂这项目）
更详细内容可以在官网中找到[usage-limits-billing-and-administration](https://docs.github.com/zh/actions/learn-github-actions/usage-limits-billing-and-administration)

##  Docker镜像构建流程
