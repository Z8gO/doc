###  SkyWalking（应用性能监控系统）
######  张煌 zhanghuang@dcits.com
###### SkyWalking: an APM(application performance monitor) system, especially designed for microservices, cloud native and container-based (Docker, Kubernetes, Mesos) architectures.
###### SkyWalking: 一个开源的可观测平台, 用于从服务和云原生基础设施收集, 分析, 聚合及可视化数据。SkyWalking 提供了一种简便的方式来清晰地观测分布式系统, 甚至横跨多个云平台。SkyWalking 更是一个现代化的应用程序性能监控(Application Performance Monitoring)系统, 尤其专为云原生、基于容器的分布式系统设计

skywalking-quick-start:  

>  https://skywalking.apache.org/zh/blog/2020-04-19-skywalking-quick-start.html

文档：  
>https://github.com/SkyAPM/document-cn-translation-of-skywalking/blob/master/docs/zh/8.0.0/README.md


####  环境搭建：
    win10  apache-skywalking-apm-es7-6.6.0 + elasticsearch-7.3.2
    
    下载地址：
        ES: https://blog.csdn.net/weixin_37281289/article/details/101483434  
        linux: https://pan.baidu.com/s/1WfS9iWnayaNVQgbC29E13g  提取码：qijm   
        windows: https://pan.baidu.com/s/1rwRAIkmssg98NX0xwY4iJA   提取码：bofl   
        SW: https://skywalking.apache.org/zh/downloads/
        选6.6.0 下的Binary Distribution for ElasticSearch 7 (Windows)

ES中的配置（config\elasticsearch.yml）：

    cluster.name: elasticsearch  
    node.name: elasticsearch-node-1  
    network.host: 127.0.0.1  
    http.port: 9200  
    discovery.seed_hosts: ["127.0.0.1"]  
    cluster.initial_master_nodes: ["elasticsearch-node-1"]


单机SW中配置(conf/application.yml):
    
    1、注释掉storage节点下 h2节点的所有配置。
    2、修改elasticsearch7节点下的配置。
    需要注意：nameSpace 中配置的是ES中配置的 cluster.name 的值
    storage:
      elasticsearch7:
        nameSpace: ${SW_NAMESPACE:"elasticsearch"}
        clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:127.0.0.1:9200}
        protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"http"}
        trustStorePath: ${SW_SW_STORAGE_ES_SSL_JKS_PATH:"../es_keystore.jks"}
        trustStorePass: ${SW_SW_STORAGE_ES_SSL_JKS_PASS:""}
    #    user: ${SW_ES_USER:""}
    #    password: ${SW_ES_PASSWORD:""}
        indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:2}
        indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:0}
        # Those data TTL settings will override the same settings in core module.
        recordDataTTL: ${SW_STORAGE_ES_RECORD_DATA_TTL:7} # Unit is day
        otherMetricsDataTTL: ${SW_STORAGE_ES_OTHER_METRIC_DATA_TTL:45} # Unit is day
        monthMetricsDataTTL: ${SW_STORAGE_ES_MONTH_METRIC_DATA_TTL:18} # Unit is month
        # Batch process setting, refer to https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/java-docs-bulk-processor.html
        bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:1000} # Execute the bulk every 1000 requests
        flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10 seconds whatever the number of requests
        concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
        resultWindowMaxSize: ${SW_STORAGE_ES_QUERY_MAX_WINDOW_SIZE:10000}
        metadataQueryMaxSize: ${SW_STORAGE_ES_QUERY_MAX_SIZE:5000}
        segmentQueryMaxSize: ${SW_STORAGE_ES_QUERY_SEGMENT_SIZE:200}


### 集群SW配置注意点

config/application.yml ：

    cluster:
      selector: ${SW_CLUSTER:nacos}  #如果使用nacos作为服务注册中心，需要有nacos相关的配置。
      nacos:
        serviceName: ${SW_SERVICE_NAME:"SkyWalking_OAP_Cluster"}
        hostPort: ${SW_CLUSTER_NACOS_HOST_PORT:localhost:8848,localhost:8849,localhost:8850}
        # Nacos Configuration namespace
        namespace: ${SW_CLUSTER_NACOS_NAMESPACE:"public"}
    storage:
      selector: ${SW_STORAGE:elasticsearch7}
      elasticsearch7:
          nameSpace: ${SW_NAMESPACE:"elasticsearch"}
    
    #下面这块如果是一台集群部署集群需要注意 restPort 和 gRPCPort
    #多台机器部署集群的话，如果确定端口没有被其他服务占用，则可以忽略
    core:
      selector: ${SW_CORE:default}
      default:
        # Mixed: Receive agent data, Level 1 aggregate, Level 2 aggregate
        # Receiver: Receive agent data, Level 1 aggregate
        # Aggregator: Level 2 aggregate
        role: ${SW_CORE_ROLE:Mixed} # Mixed/Receiver/Aggregator
        #REST 请求地址 webapp模块 使用
        restHost: ${SW_CORE_REST_HOST:0.0.0.0}
        #REST 请求端口 webapp模块 使用
        restPort: ${SW_CORE_REST_PORT:12800}
        restContextPath: ${SW_CORE_REST_CONTEXT_PATH:/}
        #gRPC请求地址 agent模块 使用
        gRPCHost: ${SW_CORE_GRPC_HOST:0.0.0.0}
        #gRPC 请求端口 agent模块 使用
        gRPCPort: ${SW_CORE_GRPC_PORT:11800}

agent/config/agent.config

        # Backend service addresses.
        collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:127.0.0.1:11800,127.0.0.1:11801,127.0.0.1:11802}
        
        
webapp/wabapp.yml

        listOfServers: 127.0.0.1:12800,127.0.0.1:12801,127.0.0.1:12802



告警规则配置（config/alarm-settings.yml）

        rules:
          #所有的规则名称必须以`_rule`结尾。
          # Rule unique name, must be ended with `_rule`.
          #最近3分钟内服务平均响应时间超过1秒。
          #服务响应时间规则
          service_resp_time_rule:
            metrics-name: service_resp_time
            #和阈值比较的符号
            op: ">"
            #阈值
            threshold: 1000
            period: 10
            #告警触发次数，触发n次后进行告警
            count: 3
            silence-period: 5
            #页面提示信息
            message: Response time of service {name} is more than 1000ms in 3 minutes of last 10 minutes.
            
          #最近2分钟的服务成功率低于80％。
          service_sla_rule:
            # Metrics value need to be long, double or int
            metrics-name: service_sla
            op: "<"
            threshold: 8000
            # The length of time to evaluate the metrics
            period: 10
            # How many times after the metrics match the condition, will trigger alarm
            count: 2
            # How many times of checks, the alarm keeps silence after alarm triggered, default as same as period.
            silence-period: 3
            message: Successful rate of service {name} is lower than 80% in 2 minutes of last 10 minutes
          #在过去3分钟内，服务90％的响应时间超过1秒
          service_resp_time_percentile_rule:
            # Metrics value need to be long, double or int
            metrics-name: service_percentile
            op: ">"
            threshold: 1000,1000,1000,1000,1000
            period: 10
            count: 3
            silence-period: 5
            message: Percentile response time of service {name} alarm in 3 minutes of last 10 minutes, due to more than one condition of p50 > 1000, p75 > 1000, p90 > 1000, p95 > 1000, p99 > 1000
          #最近2分钟内服务实例的平均响应时间超过1秒。
          service_instance_resp_time_rule:
            metrics-name: service_instance_resp_time
            op: ">"
            threshold: 1000
            period: 10
            count: 2
            silence-period: 5
            message: Response time of service instance {name} is more than 1000ms in 2 minutes of last 10 minutes
        #  Active endpoint related metrics alarm will cost more memory than service and service instance metrics alarm.
        #  Because the number of endpoint is much more than service and instance.
        #   最近2分钟内端点平均响应时间超过1秒。
        #  endpoint_avg_rule:
        #    metrics-name: endpoint_avg
        #    op: ">"
        #    threshold: 1000
        #    period: 10
        #    count: 2
        #    silence-period: 5
        #    message: Response time of endpoint {name} is more than 1000ms in 2 minutes of last 10 minutes
        
        #Webhook要求对等方是一个Web容器。
        #警报消息将按application/json内容类型通过HTTP发布。
        #JSON格式基于List<org.apache.skywalking.oap.server.core.alarm.AlarmMessage>
        #有以下关键信息：
        #范围(scopeId, scope): 所有范围都在org.apache.skywalking.oap.server.core.source.DefaultScopeDefine中定义。 
        #名字(name): 目标范围实体名称。 
        #实体的ID(id0): 作用域实体的ID，与名称匹配。 
        #实体的ID(id1): 作用域实体的ID1，与名称匹配。
        #规则名称(ruleName): 您在中配置的规则名称alarm-settings.yml。 
        #报警消息(alarmMessage): 报警文本消息。 
        #时间(startTime): 以毫秒为单位，介于当前时间和UTC 1970年1月1日午夜之间。 
        webhooks:
        #  - http://127.0.0.1/notify/
        #  - http://127.0.0.1/go-wechat/
        
<br/>

org.apache.skywalking.oap.server.core.alarm.AlarmMessage :
        
        @Setter(AccessLevel.PUBLIC)
        @Getter(AccessLevel.PUBLIC)
        public class AlarmMessage {
        
            public static AlarmMessage NONE = new NoAlarm();
        
            private int scopeId;   //指的是告警的范围类型（源码中有定义常量org.apache.skywalking.oap.server.core.source.DefaultScopeDefine）
            private String scope;  
            private String name; //告警服务名称
            private int id0;
            private int id1;
            private String ruleName;
            private String alarmMessage;
            private long startTime;
        
            private static class NoAlarm extends AlarmMessage {
            }
        }





如果skywalking是初次连接elasticsearch服务，启动会比较慢,要创建表。  
因为skywalking需要向es服务创建很多的index。所以在未创建完成之前，访问这个页面会是空白的。此时可以通过查看日志来判断启动是否完成，日志路径：logs/skywalking-oap-server.log。


#####  使用：

     https://www.jianshu.com/p/2e9bafe6edbb

#####  github源码下载  

    https://github.com/apache/skywalking.git  
    如果下载太慢或者不行的话，可以通过 gitee,先在gitee 下把GitHub的仓库克隆一份，然后下载，这样就变成国内的网络下载了。

     
#####  源码编译：

    https://www.cnblogs.com/terryMe/p/12251378.html  
    编译时间很长............

    我自己在源码编译的过程中遇到的问题：  
    下载下来的源码报错，缺失jar，但是maven并没有报错，从maven重新导入多次后仍没有解决，正常来说源码不可能有问题，那么只能是我自己的环境有问题。  
    最终解决办法：直接删除本地的maven用来存储下载的jar的文件目录，让maven重新下载所有依赖。

    之前有一个IDEA的问题，
    因为有下了开源的软件，想着只需要连接阿里的私库就可以，，就不用连接公司内网的私库了，
    所以又重新写了一个setting.xml文件，在IDEA中换了此文件后，依然还是会去公司内网的库去下载，
    IDEA中点击 File → Settings →Build Tools → Maven → Repositories 此部分为IDEA缓存的 私库的连接地址。
    结果看到了之前配置的信息，等于是此次更换setting.xml 没生效，
    解决办法：
    清理IDEA缓存。点击 File → Invalidate Caches / Restart
        

#####  相关的文档：  

    OpenTracing:开放式分布式追踪规范  
    https://www.jianshu.com/p/d2b11c079af0
    
    概念：（Tags、 Spans、 Tracers）  
    https://opentracing.io/docs/overview/spans/  
      
    OpenTracing语义标准  
    https://github.com/opentracing-contrib/opentracing-specification-zh/blob/master/specification.md

#####  SkyingWalking Plugin Development Guide  
######  SkyingWalking 插件开发
    
    英文文档：  
    https://github.com/apache/skywalking/blob/v7.0.0/docs/en/guides/Java-Plugin-Development-Guide.md  
    中文文档：
    https://skyapm.github.io/document-cn-translation-of-skywalking/zh/8.0.0/guides/Java-Plugin-Development-Guide.html  

    

######  源码学习总结：
    注意点：
    一、agent包在打包时有Premain-Class 项。
    Premain-Class: org.apache.skywalking.apm.agent.SkyWalkingAgent
    mvn  打包命令： mvnw clean package -DskipTests -Pagent 
    二、如何使用：
    通过在VM options处输入 -javaagent:/path/skywalking-agent/skywalking-agent.jar命令，进行启动。
    skywalking 插件加载过程:
    1、在启动时首先加载 skywalking-agent.jar 中的SkyWalkingAgent.class ，调用SkyWalkingAgent#premain方法  
    2、加载所有的 skywalking-plug.def文件。这个文件就是我们自定义插件中必须配置的文件。  
    
    
    看到SW在第一次启动时会去创建一大堆的表,大概分析源码是怎么去启动并第一次创建表的。（以server-starter-es7 启动为例）
    oap-server/server-starter-es7下：
    
    org.apache.skywalking.oap.server.starter.OAPServerStartUp 的main方法调用 OAPServerBootstrap.start()
    进入start()，在加载完applicationConfiguration之后，
    进行 manager.init(applicationConfiguration);  
    
    以下为init方法的源码：
    public void init(
        ApplicationConfiguration applicationConfiguration) throws ModuleNotFoundException, ProviderNotFoundException, ServiceNotProvidedException, CycleDependencyException, ModuleConfigException, ModuleStartException {
        String[] moduleNames = applicationConfiguration.moduleList();
        // SPI机制查找所有模块实现类 ModuleDefine
        ServiceLoader<ModuleDefine> moduleServiceLoader = ServiceLoader.load(ModuleDefine.class);
        // SPI机制查实模块提供者实现类 ModuleProvider
        ServiceLoader<ModuleProvider> moduleProviderLoader = ServiceLoader.load(ModuleProvider.class);
        // 配置文件定义的模块
        LinkedList<String> moduleList = new LinkedList<>(Arrays.asList(moduleNames));
        // 检查配置文件中定义的模块，系统是是否存在
        for (ModuleDefine module : moduleServiceLoader) {
            for (String moduleName : moduleNames) {
                if (moduleName.equals(module.name())) {
                    ModuleDefine newInstance;
                    try {
                        // 反射创建实例
                        newInstance = module.getClass().newInstance();
                    } catch (InstantiationException | IllegalAccessException e) {
                        throw new ModuleNotFoundException(e);
                    }
                    //  模块准备流程
                    newInstance.prepare(this, applicationConfiguration.getModuleConfiguration(moduleName), moduleProviderLoader);
                    loadedModules.put(moduleName, newInstance);
                    moduleList.remove(moduleName);
                }
            }
        }
        // Finish prepare stage
        isInPrepareStage = false;
        
        // 存在不知名module配置则抛异常
        if (moduleList.size() > 0) {
            throw new ModuleNotFoundException(moduleList.toString() + " missing.");
        }

        BootstrapFlow bootstrapFlow = new BootstrapFlow(loadedModules);
        
        // 模块加载完成 启动服务
        bootstrapFlow.start(this);
        bootstrapFlow.notifyAfterCompleted();
    }
    
        
        bootstrapFlow.start(this);
        源码：
        
        void start( ModuleManager moduleManager) throws ModuleNotFoundException, ServiceNotProvidedException, ModuleStartException {
            for (ModuleProvider provider : startupSequence) {
                String[] requiredModules = provider.requiredModules();
                if (requiredModules != null) {
                    for (String module : requiredModules) {
                        if (!moduleManager.has(module)) {
                            throw new ModuleNotFoundException(module + " is required by " + provider.getModuleName() + "." + provider
                                .name() + ", but not found.");
                        }
                    }
                }
                logger.info("start the provider {} in {} module.", provider.name(), provider.getModuleName());
                provider.requiredCheck(provider.getModule().services());
                
                /**provider 启动
                 *因为 oap-server/server-storage-plugin/storage-elasticsearch7-plugin
                 *下的resource/META-INF/services/org.apache.skywalking.oap.server.library.module.ModuleProvider 文件中配置了
                 *org.apache.skywalking.oap.server.storage.plugin.elasticsearch7.StorageModuleElasticsearch7Provider
                 *所以根据 Java SPI机制,这个等于是调用
                 *org.apache.skywalking.oap.server.storage.plugin.elasticsearch7.StorageModuleElasticsearch7Provider.start()
                 *后面就开始比如检查表是否存在不存在则创建表，初始化数据
                 */
                provider.start();
            }
        }
    


#### SkyWalking + Prometheus + Grafana

    参考文档:
    SkyWalking/docs/en/setup/backend/backend-telemetry.md

<br/>

 1、SkyWalking 配置
 
    关键配置：selector: ${SW_TELEMETRY:so11y}
    
    具体配置如下：
 
    telemetry:
      selector: ${SW_TELEMETRY:so11y}
      none:
      prometheus:
        host: ${SW_TELEMETRY_PROMETHEUS_HOST:0.0.0.0}
        port: ${SW_TELEMETRY_PROMETHEUS_PORT:1234}
      so11y:
        prometheusExporterEnabled: ${SW_TELEMETRY_SO11Y_PROMETHEUS_ENABLED:true}
        prometheusExporterHost: ${SW_TELEMETRY_PROMETHEUS_HOST:0.0.0.0}
        prometheusExporterPort: ${SW_TELEMETRY_PROMETHEUS_PORT:1234} 
        
  
  配置后启动 SkyWalking . 访问 127.0.0.1:1234 
  
  2、 Prometheus 配置
  
    在scrape_configs 下增加：
    
      - job_name: 'server'
        static_configs:
          - targets: ['localhost:1234']
         
 
  配置后启动 Prometheus，可以在 菜单的 Status → Targets 下看到服务信息。


  3、Grafana
    
    1、在Grafana中只需要配置了Prometheus 数据源。
    2、将参考文档中的json 导入到Dashboard中，即可看到SkyWalking 的指标数据。



##### SkyWalking 和 Zipkin 对比

    https://juejin.im/post/5a7a9e0af265da4e914b46f1#heading-23
    

|  类别   |  Zipkin  |  SkyWalking  |
|  ----  |  ----  |  ----  |
| 实现方式  | 拦截请求，发送（HTTP，mq）数据至zipkin服务 | java探针，字节码增强 |
| 接入方式  | 基于linkerd或者sleuth方式，引入配置即可 | javaagent字节码 |
| agent到collector的协议  | http,MQ | gRPC |
| OpenTracing  | 支持 | 支持 |
| 颗粒度  | 接口级 | 方法级 |
| 全局调用统计  | 不支持 | 支持 |
| traceid查询  | 支持 | 支持 |
| 报警 | 不支持 | 支持 |
| JVM监控 | 不支持 | 支持 |
| 健壮度 | ☆☆ | ☆☆☆☆ |
| 数据存储 | 	ES，mysql,Cassandra,内存 | ES，H2,mysql，influxdb |


##### apdex
    http://skywalking.apache.org/zh/blog/2020-07-26-apdex-and-skywalking.html
    
    
    Satisfied request : SkyWalking默认 Satisfied request 的阈值(threshold)为:500ms。
    
    Tolerating requests ：默认为 Satisfied request 阈值的四倍 即为 2000ms
    
    ApdexScore = ( (Satisfied request) + (Tolerating requests /2) ) / Total number of requests
    
                       满意请求数 +  ( 容忍请求数 / 2 )
    Apdex 得分  =  ——————————————————————————————————————————
                                 总请求数

##### SkyWalking 仪表盘中的名词：
    
    
    所有的展示数据是页面右下角的时间区间作为条件进行统计。
    Service Dashboard：
        Global:
            Global Heatmap： 系统全局耗时时间
            Global Response Time Percentile : 全局响应时间。 数值解释： p50:A p75:B p90:C p95:D (ABCD均为数字单位ms) 表示：有50%的请求低于A ms.75%低于B ms 
            Global Brief: 全局信息明细。
            Global Top Slow Endpoint: 全局最慢的拦截点排名。
        Service： （下拉框可选服务）
            Service Avg ApdexScore: 服务平均体验度得分
            Service Avg ResponseTime: 服务平均响应时间
            Service Avg Throughput : 系统平均每分钟处理请求的数量（单位：cpm）
            Service Avg SLA: 成功率。对于HTTP，表示响应为200的请求.(响应为200的请求数量除以总请求数量)
            Service ApdexScore: 服务体验度得分走势    ApdexScore = ( (Satisfied request) + (Tolerating requests /2) ) / Total number of requests
            Service ResponseTime: 服务响应时间走势
            Service Throughput : 系统处理请求的数量走势
            Service Response Time Percentile: 服务响应时间百分比走势。数值解释：p50:A p75:B p90:C p95:D (ABCD均为数字单位ms) 表示：有50%的请求低于A ms.75%低于B ms 
            Service Slow Endpoint : 响应慢的拦截点。
            Running ServiceInstance : 运行的服务实例
        Endpoint ：
            Endpoint Avg ResponseTime :  端点平均响应时间
            Endpoint Avg Throughput : 端点平均每分钟处理请求的数量（单位：cpm）
            Endpoint Avg SLA: 平均成功率。对于HTTP，表示响应为200的请求.(响应为200的请求数量除以总请求数量)
            Endpoint ResponseTime：服务响应时间走势
            Endpoint Throughput ：系统处理请求的数量走势
            Endpoint SLA：成功率。对于HTTP，表示响应为200的请求.(响应为200的请求数量除以总请求数量)
            Endpoint Response Time Percentile ：服务响应时间百分比走势。数值解释：p50:A p75:B p90:C p95:D (ABCD均为数字单位ms) 表示：有50%的请求低于A ms.75%低于B ms 
            Dependency Map : 依赖关系图
            Slow Traces: 响应时间长的调用链排序
        Instance：
            Instance Info :实例信息
            Instance Avg ResponseTime ：实例平均响应时间
            Instance Avg Throughput ：实例平均每分钟处理请求的数量（单位：cpm）
            Instance Avg SLA ：实例平均响应成功率。对于HTTP，表示响应为200的请求.(响应为200的请求数量除以总请求数量)
            Instance Throughput：实例处理请求的数量走势
            Instance SLA：实例响应成功率。对于HTTP，表示响应为200的请求.(响应为200的请求数量除以总请求数量)
            Instance ResponseTime：实例响应时间走势
            JVM Heap (MB)：JVM堆大小
            JVM Non-Heap (MB)：JVM非堆大小
            JVM GC (ms)：GC时间
            JVM GC Count：GC次数。 YoungGC Count、OldGC Count
            JVM CPU (%)：JVM 占用CPU百分比
            CLR CPU (%)：CLR 占用CPU百分比
            CLR GC (Count)：GC次数。 clrGCGen0、clrGCGen1、clrGCGen2
            CLR HeapMemory (MB)：CLR 堆内存
    Database Dashboard
        Database
            Database Avg ResponseTime：DB平均响应时间
            Database ResponseTime：DB响应时间走势图
            Database Avg Throughput：DB平均每分钟处理请求的数量（单位：cpm）
            Database Throughput：DB处理请求的数量（单位：cpm）走势图
            Database SLA：DB处理请求成功率。对于HTTP，表示响应为200的请求.(响应为200的请求数量除以总请求数量)
            Database Avg SLA：DB 处理请求的成功率。对于HTTP，表示响应为200的请求.(响应为200的请求数量除以总请求数量)
            Database Response Time Percentile：DB响应时间百分比走势。数值解释：p50:A p75:B p90:C p95:D (ABCD均为数字单位ms) 表示：有50%的请求低于A ms.75%低于B ms 
            Database TopN Records：DBTopN记录



#### SkyWalking  性能调优：

    copy From  https://cloud.tencent.com/developer/article/1563962
    
    
    如果你正在使用SkyWalking作为分布式跟踪系统，而且是使用elasticsearch作为存储引擎，那么这篇文章中针对SkyWalking的优化你不妨看一下，说不定就有用了呢？
    
    OAP优化
    skywalking写入ES的操作是使用了ES的批量写入接口，我们要做的是调整相关参数尽量降低ES索引的写入频率。参数调整主要是针对skywalking的配置文件application.yml，相关参数如下：
    
    storage:
      elasticsearch:
        bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:4000} # Execute the bulk every 2000 requests
        bulkSize: ${SW_STORAGE_ES_BULK_SIZE:40} # flush the bulk every 20mb
        flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:30} # flush the bulk every 10 seconds whatever the number of requests
        concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:4} # the number of concurrent requests
        metadataQueryMaxSize: ${SW_STORAGE_ES_QUERY_MAX_SIZE:8000}
        
    调整bulkActions默认2000次请求批量写入一次改到4000次；
    bulkSize批量刷新从20M一次到40M一次；
    flushInterval每10秒刷新一次堆改为每30秒刷新；
    concurrentRequests查询的最大数量由5000改为8000。



