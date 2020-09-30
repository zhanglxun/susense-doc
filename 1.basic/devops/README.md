## 流水线



### pipline 

目前，使用了如下工具，执行开发到部署的自动化流水线；



<img src="http://img.susense.cn/devops19.jpg" style="zoom:50%;" />



### 工作步骤

- 工程师在本地使用Git工具，拉取代码仓库的代码；
- 在本地开发测试自测后，将代码推送到GitLab 仓库；
- 手动触发
  - 在Jenkins中配置各微服务的相关服务配置
  - 通过不同环境、角色和权限的隔离，可以分别触发测试环境、预览环境、和生产环境的构建，打包，部署和服务重启
- 自动触发
  - 通过编写gitlab 中的yaml 脚本，触发CI/CD 流程
  - 通过若干步骤，执行包括：代码扫描，编译，构建，打包，制作镜像，推送镜像仓库，拉取镜像执行等自动化的一系列流程；



## 相关工具（预览）



### portal

<img src="http://img.susense.cn/image-20200919183243582.png" alt="image-20200919183243582" style="zoom:50%;" />



### GitLab

> Gitlab 一个开源的git仓库管理平台，方便团队协作开发、管理。在GitLab上可以实现完整的CI（持续集成）、CD（持续发布）流程。而且还提供了免费使用的Plan，以及免费的可以独立部署的社区版本。



> 系统预览

<img src="http://img.susense.cn/image-20200919183402327.png" alt="image-20200919181225222" style="zoom:50%;" />

<img src="http://img.susense.cn/image-20200919183603755.png" alt="image-20200919183603755" style="zoom:50%;" />



### jenkins

> Jenkins 是一个开源项目，提供了一种易于使用的持续集成系统，使开发者从繁杂的集成中解脱出来，专注于更为重要的业务逻辑实现上。 ... 持续集成是一种软件开发实践，对于提高软件开发效率并保障软件开发质量提供了理论基础。 Jenkins 是一个开源软件项目，旨在提供一个开放易用的软件平台，使持续集成变成可能。

> 系统预览

<img src="http://img.susense.cn/image-20200919182959422.png" alt="image-20200919182959422" style="zoom:50%;" />

### Sonatype nexus

>  Nexus是Maven仓库管理器，也可以叫Maven的私服。Nexus是一个强大的Maven仓库管理器，它极大地简化了自己内部仓库的维护和外部仓库的访问。利用Nexus你可以只在一个地方就能够完全控制访问和部署在你所维护仓库中的每个Artifact。Nexus是一套“开箱即用”的系统不需要数据库，它使用文件系统加Lucene来组织数据。

> 系统预览

<img src="http://img.susense.cn/image-20200919182242327.png" alt="image-20200919182242327" style="zoom:50%;" />

### Apollo 配置中心

> 随着程序功能的日益复杂，程序的配置日益增多：各种功能的开关、参数的配置、服务器的地址……
>
> 对程序配置的期望值也越来越高：配置修改后实时生效，灰度发布，分环境、分集群管理配置，完善的权限、审核机制……
>
> 在这样的大环境下，传统的通过配置文件、数据库等方式已经越来越无法满足开发人员对配置管理的需求。
>
> Apollo配置中心应运而生！



Apollo（阿波罗）是携程框架部门研发的开源配置管理中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性。

Apollo支持4个维度管理Key-Value格式的配置：

1. application (应用)
2. environment (环境)
3. cluster (集群)
4. namespace (命名空间)

同时，Apollo基于开源模式开发，开源地址:

```html
https://github.com/ctripcorp/apollo
```

<img src="http://img.susense.cn/image-20200919184300923.png" alt="image-20200919184300923" style="zoom:50%;" />



### harbor 镜像仓库

> Harbor 仓库

![Harbor](http://img.susense.cn/0a5692d8d5af73c66ab1d54d7788890b.png)

### sonarQube

> Sonaqube

![image-20200919182805102](http://img.susense.cn/image-20200919182805102.png)



### kubernates

> Kubernetes，是一个全新的基于容器技术的分布式架构领先方案。Kubernetes(k8s)是Google开源的容器集群管理系统（谷歌内部:Borg）。在Docker技术的基础上，为容器化的应用提供部署运行、资源调度、服务发现和动态伸缩等一系列完整功能，提高了大规模容器集群管理的便捷性。



> Kubernetes是一个完备的分布式系统支撑平台，具有完备的集群管理能力，多扩多层次的安全防护和准入机制、多租户应用支撑能力、透明的服务注册和发现机制、內建智能负载均衡器、强大的故障发现和自我修复能力、服务滚动升级和在线扩容能力、可扩展的资源自动调度机制以及多粒度的资源配额管理能力。同时Kubernetes提供完善的管理工具，涵盖了包括开发、部署测试、运维监控在内的各个环节。



> 系统预览

<img src="http://img.susense.cn/image-20200919183814433.png" alt="image-20200919183814433" style="zoom:50%;" />

<img src="http://img.susense.cn/image-20200919183850779.png" alt="image-20200919183850779" style="zoom:50%;" />

## 注意事项

> 此部分偏运维层面，但是作为研发效能的基础实施，此上述部分能大幅度提高效能。

