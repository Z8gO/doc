######张煌 zhanghuang@dcits.com

##InfluxDB 学习笔记
####时序数据库

######场景
平台需要监控各种状态，各种细节的数据如：内存、CPU、响应时间、用户操作等等，并且需要把每时每刻监控的数据记录下来，用来做大数据分析。如果只是存储下来不查询也还好（虽然已经是不小的成本），但如果需要快速查询“某个时间点的系统的各个细节数据信息”这样的多纬度分组聚合查询，那么时序数据库会是一个很好的选择。

######概念相关参考：

 > <a href="https://www.jianshu.com/p/f0905f36e9c3" target="_blank">InfluxDB 入门</a>  
<a href="https://www.influxdata.com/" target="_blank">influxdb官网</a>   
<a href="https://docs.influxdata.com/influxdb/v1.7/" target="_blank">相关API官网</a>  


######操作语句： 
######请注意语句区分大小写  

>show databases;

>create database Test;  

>use Test;


>在InfluxDB当中，并没有表（table）这个概念，取而代之的是MEASUREMENTS，MEASUREMENTS的功能与传统数据库中的表一致，因此我们也可以将MEASUREMENTS称为InfluxDB中的表。  

>show measurements;  

>需要注意的是，influxdb不需要像传统数据库一样创建各种表，其表的创建主要是通过第一次数据插入时自动创建，如下：  
insert mytest, server=serverA count=1,name=serverA //自动创建表  
“mytest”，“server” 是 tags，“count”、“name” 是 fields
fields 中的 value 基本不用于索引  







<br/><br/><br/><br/><br/><br/><br/><br/>
