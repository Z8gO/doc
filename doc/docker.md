
### Docker

##### DockerFile

   简单的DockerFile demo

    FROM  java:8
    ADD plays-0.0.1   /app/plays-0.0.1 
    WORKDIR /app/plays-0.0.1/
    EXPOSE  31027
    ENTRYPOINT  ["java","-Dspring.config.location=conf/application.properties","-Djava.ext.dirs=lib/","-jar","lib/plays-0.0.1-SNAPSHOT.jar"," &"]


<br/>

>注意点：  
>1、DockerFile中的 ADD ： 增加的文件夹在容器中也必须有。 因此必须写成  ADD dirName  /targetPath/dirName     
>2、 WORKDIR 容器启动时的工作目录  
>3、 EXPOSE 31027 服务使用容器的端口  
>4、 执行命令

