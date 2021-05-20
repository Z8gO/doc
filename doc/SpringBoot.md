

#  Spring Boot 学习笔记

配置文件:  
>application-dev.properties: 开发环境  
application-test.properties: 测试环境  
apptication-prod.properties: 生产环境

启动时使用测试环境配置文件  
>java -jar xxx.jar --spring.profiles.active=test  

服务端口设置为 8099  
> java -jar xxx.jar --server.port=8099  


 <br/><br/><br/><br/><br/><br/>
