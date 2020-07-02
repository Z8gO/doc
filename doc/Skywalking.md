###  skyWalking（应用性能监控系统） 
######  张煌 zhanghuang@dcits.com

######  学习SW同时，可以顺带学习一下 云原生、 devops 的思想理念。


skywalking-quick-start：
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


SW中配置(conf/application.yml):
    
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





如果skywalking是初次连接elasticsearch服务，启动会比较慢，（很慢。。。）  
因为skywalking需要向es服务创建很多的index。所以在未创建完成之前，访问这个页面会是空白的。此时可以通过查看日志来判断启动是否完成，日志路径：logs/skywalking-oap-server.log。


#####  使用：

     https://www.jianshu.com/p/2e9bafe6edbb

github 下载  

    https://github.com/apache/skywalking.git  
    如果下载太慢或者不行的话，可以通过 gitee,先在gitee 下把GitHub的仓库克隆一份，然后下载，这样就变成国内的网络下载了。

     
#####  源码编译：

    https://www.cnblogs.com/terryMe/p/12251378.html
    



