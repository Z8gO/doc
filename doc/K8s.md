#### k8s yaml文件编写详解
##### copy from https://blog.csdn.net/qq_16754231/article/details/102460023

##### 简介
    k8s(Kubernetes)中Pod,Deployment,ReplicaSet（ReplicationController）,Service之间关系分析

    Pod：来管理容器，每个 Pod 可以包含一个或多个紧密关联的容器

    rc (ReplicationController)：来管理pod

    rs (ReplicaSet)：是rc的升级版，也是来管理pod，Kubernetes官方强烈建议避免直接使用ReplicaSet，而应该通过Deployment来创建RS和Pod。由于ReplicaSet是ReplicationController的代替物，因此用法基本相同，唯一的区别在于ReplicaSet支持集合式的selector。

    Deployment：更加方便的管理Pod和Replica Set，提供发布更新维护监控等功能

    service：是在这一整套基础之上提供给外部的稳定的服务
    

##### pod完整定义文件

    apiVersion: v1                    #必选，版本号，例如v1,版本号必须可以用 kubectl api-versions 查询到 .
    kind: Pod                            #必选，Pod
    metadata:                            #必选，元数据
      name: string                      #必选，Pod名称
      namespace: string                 #必选，Pod所属的命名空间,默认为"default"
      labels:                             #自定义标签
        - name: string                  #自定义标签名字
      annotations:                             #自定义注释列表
        - name: string
    spec:                                   #必选，Pod中容器的详细定义
      containers:                           #必选，Pod中容器列表
      - name: string                          #必选，容器名称,需符合RFC 1035规范
        image: string                         #必选，容器的镜像名称
        imagePullPolicy: [ Always|Never|IfNotPresent ]  #获取镜像的策略 Alawys表示下载镜像 IfnotPresent表示优先使用本地镜像,否则下载镜像，Nerver表示仅使用本地镜像
        command: [string]                 #容器的启动命令列表，如不指定，使用打包时使用的启动命令
        args: [string]                       #容器的启动命令参数列表
        workingDir: string                     #容器的工作目录
        volumeMounts:                     #挂载到容器内部的存储卷配置
        - name: string                    #引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
          mountPath: string                 #存储卷在容器内mount的绝对路径，应少于512字符
          readOnly: boolean                 #是否为只读模式
        ports:                            #需要暴露的端口库号列表
        - name: string                    #端口的名称
          containerPort: int                #容器需要监听的端口号
          hostPort: int                      #容器所在主机需要监听的端口号，默认与Container相同
          protocol: string                  #端口协议，支持TCP和UDP，默认TCP
        env:                                #容器运行前需设置的环境变量列表
        - name: string                      #环境变量名称
          value: string                     #环境变量的值
        resources:                            #资源限制和请求的设置
          limits:                           #资源限制的设置
            cpu: string                     #Cpu的限制，单位为core数，将用于docker run --cpu-shares参数
            memory: string                  #内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
          requests:                           #资源请求的设置
            cpu: string                     #Cpu请求，容器启动的初始可用数量
            memory: string                    #内存请求,容器启动的初始可用数量
        livenessProbe:                      #对Pod内各容器健康检查的设置，当探测无响应几次后将自动重启该容器，检查方法有exec、httpGet和tcpSocket，对一个容器只需设置其中一种方法即可
          exec:                           #对Pod容器内检查方式设置为exec方式
            command: [string]               #exec方式需要制定的命令或脚本
          httpGet:                        #对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port
            path: string
            port: number
            host: string
            scheme: string
            HttpHeaders:
            - name: string
              value: string
          tcpSocket:                  #对Pod内个容器健康检查方式设置为tcpSocket方式
             port: number
           initialDelaySeconds: 0       #容器启动完成后首次探测的时间，单位为秒
           timeoutSeconds: 0            #对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
           periodSeconds: 0             #对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
           successThreshold: 0
           failureThreshold: 0
           securityContext:
             privileged: false
        restartPolicy: [Always | Never | OnFailure] #Pod的重启策略，Always表示一旦不管以何种方式终止运行，kubelet都将重启，OnFailure表示只有Pod以非0退出码退出才重启，Nerver表示不再重启该Pod
        nodeSelector: obeject           #设置NodeSelector表示将该Pod调度到包含这个label的node上，以key：value的格式指定
        imagePullSecrets:             #Pull镜像时使用的secret名称，以key：secretkey格式指定
        - name: string
        hostNetwork: false              #是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
        volumes:                        #在该pod上定义共享存储卷列表
        - name: string                  #共享存储卷名称 （volumes类型有很多种）
          emptyDir: {}                  #类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
          hostPath: string              #类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
            path: string                  #Pod所在宿主机的目录，将被用于同期中mount的目录
          secret:                       #类型为secret的存储卷，挂载集群与定义的secre对象到容器内部
            scretname: string  
            items:     
            - key: string
              path: string
          configMap:                          #类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
            name: string
            items:
            - key: string
              path: string
    


##### K8s的pod的理解

     pod共享相同的IP地址和端口空间。
     这意味着在同一 pod中的容器运行的 多个进程需要注意不能绑定到相同的端口号， 否则会导致端口冲突，
     但这只涉及同一pod中的容器。 由于每个pod都有独立的端口空间， 对于不同 pod中的容器来说 则永远不会遇到端口冲突
     一个 pod中的所有容器也都具有相同的loopback 网络接口， 因此容器可以通过localhost 与同一 pod中的其他容器进行通信。
     
     pod中的容器
     k8s中的思想是：每个容器只安装一个进程，然后多个或一个容器属于一个pod。然后这个pod下的容器可以通过volume的方式共享磁盘。
     也就是说，应该把整个pod看作虚拟机，然后每个容器相当于运行在虚拟机的进程。
     
     将多层应用分散到多个 pod 中
     虽然可以把多个容器放在同一个pod下，但是应该根据应用将容器分布到不同的pod中。原因如下：
     
     每个pod是部署再固定的node上的，将前端、后端应用部署在同一个pod里发挥不出集群的作用
     当需要进行节点扩容的时候，如果前后端在一个pod里，扩容一个pod后就有两个前端、后端，这样多出的前端是没有意义的而且很有难度
     何时在pod使用多个容器
     一般情况下建议单容器pod
     多容器需要同时扩缩容，是否必须一起运行，代表的是一个主体还是多个独立的组件


##### 完整deployment

 1.定义Deployment来创建Pod和ReplicaSet

 2.滚动升级和回滚应用

 3.扩容和缩容

 4.暂停和继续Deployment  
 
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata: <Object>
    spec: <Object>
      minReadySeconds: <integer>                        #设置pod准备就绪的最小秒数
      paused: <boolean>                                 #表示部署已暂停并且deploy控制器不会处理该部署
      progressDeadlineSeconds: <integer>
      strategy: <Object>                                #将现有pod替换为新pod的部署策略
        rollingUpdate: <Object>                         #滚动更新配置参数，仅当类型为RollingUpdate
          maxSurge: <string>                            #滚动更新过程产生的最大pod数量，可以是个数，也可以是百分比
          maxUnavailable: <string>                      #
        type: <string>                                  #部署类型，Recreate，RollingUpdate
      replicas: <integer>                               #pods的副本数量
      selector: <Object>                                #pod标签选择器，匹配pod标签，默认使用pods的标签
        matchLabels: <map[string]string> 
          key1: value1
          key2: value2
        matchExpressions: <[]Object>
          operator: <string> -required-                 #设定标签键与一组值的关系，In, NotIn, Exists and DoesNotExist
          key: <string> -required-
          values: <[]string>   
      revisionHistoryLimit: <integer>                   #设置保留的历史版本个数，默认是10
      rollbackTo: <Object> 
        revision: <integer>                             #设置回滚的版本，设置为0则回滚到上一个版本
      template: <Object> -required-
        metadata:
        spec:
          containers: <[]Object>                        #容器配置
          - name: <string> -required-                   #容器名、DNS_LABEL
            image: <string> #镜像
            imagePullPolicy: <string>                   #镜像拉取策略，Always、Never、IfNotPresent
            ports: <[]Object>
            - name:                                     #定义端口名
              containerPort:                            #容器暴露的端口
              protocol: TCP                             #或UDP
            volumeMounts: <[]Object>
            - name: <string> -required-                 #设置卷名称
              mountPath: <string> -required-            #设置需要挂载容器内的路径
              readOnly: <boolean>                       #设置是否只读
            livenessProbe: <Object>                     #就绪探测
              exec: 
                command: <[]string>
              httpGet:
                port: <string> -required-
                path: <string>
                host: <string>
                httpHeaders: <[]Object>
                  name: <string> -required-
                  value: <string> -required-
                scheme: <string> 
              initialDelaySeconds: <integer>            #设置多少秒后开始探测
              failureThreshold: <integer>               #设置连续探测多少次失败后，标记为失败，默认三次
              successThreshold: <integer>               #设置失败后探测的最小连续成功次数，默认为1
              timeoutSeconds: <integer>                 #设置探测超时的秒数，默认1s
              periodSeconds: <integer>                  #设置执行探测的频率（以秒为单位），默认1s
              tcpSocket: <Object>                       #TCPSocket指定涉及TCP端口的操作
                port: <string> -required-               #容器暴露的端口
                host: <string>                          #默认pod的IP
            readinessProbe: <Object>                    #同livenessProbe
            resources: <Object>                         #资源配置
              requests: <map[string]string>             #最小资源配置
                memory: "1024Mi"
                cpu: "500m"                             #500m代表0.5CPU
              limits: <map[string]string>               #最大资源配置
                memory:
                cpu:         
          volumes: <[]Object>                           #数据卷配置
          - name: <string> -required-                   #设置卷名称,与volumeMounts名称对应
            hostPath: <Object>                          #设置挂载宿主机路径
              path: <string> -required- 
              type: <string>                            #类型：DirectoryOrCreate、Directory、FileOrCreate、File、Socket、CharDevice、BlockDevice
          - name: nfs
            nfs: <Object>                               #设置NFS服务器
              server: <string> -required-               #设置NFS服务器地址
              path: <string> -required-                 #设置NFS服务器路径
              readOnly: <boolean>                       #设置是否只读
          - name: configmap
            configMap: 
              name: <string>                            #configmap名称
              defaultMode: <integer>                    #权限设置0~0777，默认0664
              optional: <boolean>                       #指定是否必须定义configmap或其keys
              items: <[]Object>
              - key: <string> -required-
                path: <string> -required-
                mode: <integer>
          restartPolicy: <string>                       #重启策略，Always、OnFailure、Never
          nodeName: <string>
          nodeSelector: <map[string]string>
          imagePullSecrets: <[]Object>
          hostname: <string>
          hostPID: <boolean>
    status: <Object>


eg:

    apiVersion: extensions/v1beta1   
    kind: Deployment                 
    metadata:
      name: string               #Deployment名称
    spec:
      replicas: 3 #目标副本数量
      strategy:
        rollingUpdate:  
          maxSurge: 1      #滚动升级时最大同时升级1个pod
          maxUnavailable: 1 #滚动升级时最大允许不可用的pod个数
      template:         
        metadata:
          labels:
            app: string  #模板名称
        sepc: #定义容器模板，该模板可以包含多个容器
          containers:                                                                   
            - name: string                                                           
              image: string 
              ports:
                - name: http
                  containerPort: 8080 #对service暴露端口

 
k8s如何滚动升级和回滚应用

    进行滚动升级的时候先在yaml文件中更新镜像的版本，然后根据设置需求设置maxSurge、和maxUnavailable的值即可完成

k8s如何完成扩容和缩容   

    修改replicas的值后重新发布即可
    
Service

    apiVersion: v1
    kind: Service
    matadata:                                #元数据
      name: string                           #service的名称
      namespace: string                      #命名空间
      labels:                                #自定义标签属性列表
        - name: string
      annotations:                           #自定义注解属性列表
        - name: string
    spec:                                    #详细描述
      selector: []                           #label selector配置，将选择具有label标签的Pod作为管理 范围
      type: string                           #service的类型，指定service的访问方式，默认为clusterIp
      clusterIP: string                      #虚拟服务地址
      sessionAffinity: string                #是否支持session
      ports:                                 #service需要暴露的端口列表
      - name: string                         #端口名称
        protocol: string                     #端口协议，支持TCP和UDP，默认TCP
        port: int                            #服务监听的端口号
        targetPort: int                      #需要转发到后端Pod的端口号
        nodePort: int                        #当type = NodePort时，指定映射到物理机的端口号
      status:                                #当spce.type=LoadBalancer时，设置外部负载均衡器的地址
        loadBalancer:                        #外部负载均衡器
          ingress:                           #外部负载均衡器
            ip: string                       #外部负载均衡器的Ip地址值
            hostname: string                 #外部负载均衡器的主机名
            
            
            
            
            
  k8s volumes 实践

##### 配置详细：

    https://blog.csdn.net/Juwenzhe_HEBUT/article/details/89478631
    
    1. yum install -y nfs-utils
    2. mkdir -p /volumes/
    3. vim /etc/exports
       /volumes 192.168.0.0/16(rw,no_root_squash)

   
###### 配置文件注意点：

     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: nginx-vol-deploy-nfs
       namespace: default
     spec:
       selector:
         matchLabels:
           app: nginx-vol-pod-nfs
       replicas: 2
       template:
         metadata:
           labels:
             app: nginx-vol-pod-nfs
         spec:
           containers:
           - name: nginx-containers-nfs
             image: nginx
             
       #主要是下面这一块， mountPath对应容器内的路径
             volumeMounts:
             - name: html
               mountPath: /usr/share/nginx/html/
        # path 对应本地的路径
           volumes:
           - name: html
             nfs:
               path: /k8s/volumes
               server: 10.