######张煌 zhanghuang@dcits.com

##Nacos 学习笔记 
国产的注册中心、分布式配置管理：Nacos，来自阿里巴巴


######链接: https://pan.baidu.com/s/1yTvTL3M9J--n-iHhog7fzg  
######提取码: k5bb

<br/>

>解压后运行：bin 目录下的  startup.cmd  OR  startup.sh  
默认登陆用户名及密码：  nacos/ nacos

<br/>
两个主要功能:

注册中心（服务发现，即服务注册服务订阅及简单的健康管理功能）  
配置中心（类似于携程Apllo和SpringCloudConfig，更接近Apollo）

Spring Cloud 集成：  
1、注意pom中增加jar依赖  
2、配置文件bootstrap.prpperties：  

      spring.application.name=nacos-hello
      server.port=5001
      
      #################
      ##配置管理配置
      ##配置项参考：https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config
      #################
      #通过设置 spring.cloud.nacos.config.enabled = false 来完全关闭 Spring Cloud Nacos Config
      spring.cloud.nacos.config.enabled = true 
      # 注意端口不能省略
      spring.cloud.nacos.config.server-addr= 127.0.0.1:80
      #namespace 对应的 id，id 值可以在 Nacos 的控制台获取。
      spring.cloud.nacos.config.namespace= ec246752-fdc5-4203-9e25-736b38af2458
      #在没有明确指定 ${spring.cloud.nacos.config.group} 配置的情况下， 默认使用的是 DEFAULT_GROUP
      spring.cloud.nacos.config.group=TEST_GROUP
         
      #################
      ##服务注册发现配置
      ##配置项参考https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-discovery
      #################
      spring.cloud.nacos.discovery= true 
      spring.cloud.nacos.discovery.server-addr= 127.0.0.1:80
      #设置服务所处的分组
      spring.cloud.nacos.discovery.group= TEST_GROUP
      spring.cloud.nacos.discovery.namespace= ec246752-fdc5-4203-9e25-736b38af2458



Nacos 使用：

1、自己注册的服务启动后：在 服务管理—服务列表中可查看自己的服务。  
2、在 配置管理—配置列表中进行配置文件的设置。（右边的 + 号为新增配置文件）


    Data ID 格式为：
        ${prefix}-${spring.profile.active}.${file-extension}  
    不标注环境类型则不会拼接
    如：
	创建nacos-hello服务的properties类型的配置文件，  
	Data ID 默认是 nacos-hello.properties  
	生产：nacos-hello-prod.properties  
	开发：nacos-hello-dev.properties
	配置内容按照对应的需要的配置项进行配置。
	
3、@RefreshScope 在服务提供方REST请求的controller上需要增加这个注解。  
表示数据会刷新。否则若在Nacos中进行配置发布后，访问接口上的配置数据不会刷新
    


###Nacos 概念：

>Nacos 引入了一些基本的概念，系统性的了解一下这些概念可以帮助您更好的理解和正确的使用 Nacos 产品。

####地域
>物理的数据中心，资源创建成功后不能更换。

####可用区
>同一地域内，电力和网络互相独立的物理区域。同一可用区内，实例的网络延迟较低。

####接入点
>地域的某个服务的入口域名。

####命名空间
>用于进行租户粒度的配置隔离。不同的命名空间下，可以存在相同的 Group 或 Data ID 的配置。Namespace 的常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。

####配置
>在系统开发过程中，开发者通常会将一些需要变更的参数、变量等从代码中分离出来独立管理，以独立的配置文件的形式存在。目的是让静态的系统工件或者交付物（如 WAR，JAR 包等）更好地和实际的物理运行环境进行适配。配置管理一般包含在系统部署的过程中，由系统管理员或者运维人员完成。配置变更是调整系统运行时的行为的有效手段。

####配置管理
>系统配置的编辑、存储、分发、变更管理、历史版本管理、变更审计等所有与配置相关的活动。

####配置项
>一个具体的可配置的参数与其值域，通常以 param-key=param-value 的形式存在。例如我们常配置系统的日志输出级别（logLevel=INFO|WARN|ERROR） 就是一个配置项。

####配置集
>一组相关或者不相关的配置项的集合称为配置集。在系统中，一个配置文件通常就是一个配置集，包含了系统各个方面的配置。例如，一个配置集可能包含了数据源、线程池、日志级别等配置项。

####配置集 ID
>Nacos 中的某个配置集的 ID。配置集 ID 是组织划分配置的维度之一。Data ID 通常用于组织划分系统的配置集。一个系统或者应用可以包含多个配置集，每个配置集都可以被一个有意义的名称标识。Data ID 通常采用类 Java 包（如 com.taobao.tc.refund.log.level）的命名规则保证全局唯一性。此命名规则非强制。

####配置分组
>Nacos 中的一组配置集，是组织配置的维度之一。通过一个有意义的字符串（如 Buy 或 Trade ）对配置集进行分组，从而区分 Data ID 相同的配置集。当您在 Nacos 上创建一个配置时，如果未填写配置分组的名称，则配置分组的名称默认采用 DEFAULT_GROUP 。配置分组的常见场景：不同的应用或组件使用了相同的配置类型，如 database_url 配置和 MQ_topic 配置。

####配置快照
>Nacos 的客户端 SDK 会在本地生成配置的快照。当客户端无法连接到 Nacos Server 时，可以使用配置快照显示系统的整体容灾能力。配置快照类似于 Git 中的本地 commit，也类似于缓存，会在适当的时机更新，但是并没有缓存过期（expiration）的概念。

####服务
>通过预定义接口网络访问的提供给客户端的软件功能。

####服务名
>服务提供的标识，通过该标识可以唯一确定其指代的服务。

####服务注册中心
>存储服务实例和服务负载均衡策略的数据库。

####服务发现
>在计算机网络上，（通常使用服务名）对服务下的实例的地址和元数据进行探测，并以预先定义的接口提供给客户端进行查询。


####元信息
>Nacos数据（如配置和服务）描述信息，如服务版本、权重、容灾策略、负载均衡策略、鉴权配置、各种自定义标签 (label)，从作用范围来看，分为服务级别的元信息、集群的元信息及实例的元信息。


####应用
>用于标识服务提供方的服务的属性。

####服务分组
>不同的服务可以归类到同一分组。

####虚拟集群
>同一个服务下的所有服务实例组成一个默认集群, 集群可以被进一步按需求划分，划分的单位可以是虚拟集群。

####实例
>提供一个或多个服务的具有可访问网络地址（IP:Port）的进程。

####权重
>实例级别的配置。权重为浮点数。权重越大，分配给该实例的流量越大。

####健康检查
>以指定方式检查服务下挂载的实例 (Instance) 的健康度，从而确认该实例 (Instance) 是否能提供服务。根据检查结果，实例 (Instance) 会被判断为健康或不健康。对服务发起解析请求时，不健康的实例 (Instance) 不会返回给客户端。

####健康保护阈值
>为了防止因过多实例 (Instance) 不健康导致流量全部流向健康实例 (Instance) ，继而造成流量压力把健康实例 (Instance) 压垮并形成雪崩效应，应将健康保护阈值定义为一个 0 到 1 之间的浮点数。当域名健康实例 (Instance) 占总服务实例 (Instance) 的比例小于该值时，无论实例 (Instance) 是否健康，都会将这个实例 (Instance) 返回给客户端。这样做虽然损失了一部分流量，但是保证了集群的剩余健康实例 (Instance) 能正常工作。


######Nacos 升级1.3版本和1.2版本的不同

>[#2530][#2758]一致性协议抽象  
[#2560]Nacos客户端：ServerHttpAgent生成未格式化的URL  
[#2569]更改NamingProxy#reqAPI方法引发异常日志描述  
[#2577]修复命名客户端http读取超时和连接超时属性  
[#2638]支持https    
[#2647]修改配置服务md5生成方法  
[#2661]配置详细信息返回配置列表时保留查询条件  
[#2761]ProtocolManager类的getCpProtocol（）/getApProtocol（）方法的代码可以优化。  
[#2697]修复属性有空行且无法编辑的问题  
[#2738]租户合法性验证。#2785  
[#2740]修改获取远程状态的方式  
[#2664]UI添加groupName参数  
[#2765]优化MemberUtils类的kRandom方法  
[#2842]用Jackson替换Fastjson  
[#2871]配置服务器快速启动时发生SQLException  


#####Nacos 集群搭建（本地win10）
######集群信息
> Nacos 版本： 1.2.1  
端口：8848 8849 8850   
nginx-1.14 80端口代理


搭建过程：
>1、复制三份 nacos-server-1.2.1   
重命名为：nacos-server-1.2.1-8848 、nacos-server-1.2.1-8849、nacos-server-1.2.1-8850  
2、修改application.properties  
>>1、对 server.port 修改该节点对应的端口。    
2、nacos数据持久化配置（如果没有mysql环境，可以直接复制下面的配置连接）。  
连接mysql 创建数据库nacos_config 执行 conf/nacos-mysql.sql,执行完成后可看到nacos的表。  
3、mysql 配置信息：  
db.num=1  
db.url.0=jdbc:mysql://114.55.167.47:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root  
db.password=zhanghuang  

>3、复制 cluster.conf.example 重命名为 cluster.conf，并进行编辑。  
	修改内容如下：  
	172.20.10.5:8848  
	172.20.10.5:8849   
	172.20.10.5:8850  
<font color='red'>这里需要注意，ip地址不能写成127.0.0.1 否则集群有问题。</font>  

######一键启动nacos集群脚本 startNacosCluster.bat


    echo  ******************************************************************
    echo start Nacos cluster........ 
    echo  ******************************************************************

    start cmd /k E:\nacos-server-1.2.1-cluster\nacos-server-1.2.1-8848\bin\startup.cmd
    start cmd /k E:\nacos-server-1.2.1-cluster\nacos-server-1.2.1-8849\bin\startup.cmd
    start cmd /k E:\nacos-server-1.2.1-cluster\nacos-server-1.2.1-8850\bin\startup.cmd


nginx 配置：  
编辑nginx.conf 在 http 节点下增加:

	    upstream nacos {
		  server 127.0.0.1:8848;
		  server 127.0.0.1:8849;
		  server 127.0.0.1:8850;
		}

		server {
		  listen 80;
		  server_name  localhost;
		  location /nacos {
		    proxy_pass http://nacos/nacos;
		  }
		}


请注意新增server节点下的listen 配置，如已经有使用80，则更换端口， location 的配置为 /nacos不能写为 /nacos/  

启动nginx： cd 到nginx 目录下， 执行  start nginx.exe 启动nginx。  (停止  nginx.exe -s stop)

通过访问  http://localhost/nacos 则可以访问 nacos 集群。


<br/>

***
简单的入门结束
***


<br/><br/><br/><br/><br/><br/><br/><br/>
<br/><br/><br/><br/><br/><br/><br/><br/>
<br/><br/><br/><br/><br/><br/><br/><br/>