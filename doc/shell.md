###常用到的shell 命令：


##### 获取application.properties中，playsText配置项的值  

> cat application.properties | grep "playsText" |  grep -v "#" | tail -n 1 | cut -d "=" -f2- | awk '{print $1}'

#####  获取bootstrap.yml中，playsText配置项的值 

>cat bootstrap.yml | grep -w "playsText:" | grep -v "#" | awk  'NR==1{print $2}' | tr -d '\r'

    grep -v "#"  排除注释内容