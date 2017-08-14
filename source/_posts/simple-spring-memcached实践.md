title: simple-spring-memcached实践
date: 2015-09-02 09:40:02
categories: 技术
tags: [SSM,缓存]
---
前段时间，让我学习一下SSM的缓存技术，在大概了解了之后我在项目里面尝试加入SSM，下面是我完成的几个步骤。
### 安装memcached服务器
memcached的服务器网上都有的下载这里就不贴链接了，建议安装后将memcached服务设置为自动启动，同时，我的注册表MemcachedServer的ImagePath属性值设为<font color="red"> "D:\memcached\memcached.exe" -d runservice-1 127.0.0.1 -m 64 -p 12000</font>设置好后如图：<br>
<img src="/img/SSMshijian.png"/>
### memcached服务器的基本操作
#### 首先
首先保证你打开了系统的<font color="red">telnet服务</font>以及win+R->services.msc里的memcachedServer的自动启动，如下图:<br>
<img src="/img/SSMserverAuto.png"/>
#### cmd下的操作命令
cmd窗口下 telnet 127.0.0.1 11211打开memcached服务器,如果窗口进入黑屏，只有一个光标闪动就表明成功了(没有消息就是好事-。-)。
##### 存储命令
存储命令的格式:

    <command name> <key> <flags> <exptime> <bytes>    
	<data block>    

    <command name> 	set/add/replace
	<key> 			查找关键字
	<flags> 		客户机使用它存储关于键值对的额外信息
	<exptime> 		该数据的存活时间，0表示永远
	<bytes> 		存储字节数
	<data block> 	存储的数据块（可直接理解为key-value结构中的value)    

1. 添加
<img src="/img/SSM1.png"/>
这个set的命令在memcached中的使用频率极高。set命令不但可以简单添加，如果set的key已经存在，该命令可以更新该key所对应的原来的数据，也就是实现更新的作用。
可以通过“get 键名”的方式查看添加进去的记录：
<img src ="/img/SSM2.png"/>
类推，我们知道肯定有delete <name>命令。
至于add命令是只有在数据不存在时才能添加的，还有replace命令只有在数据存在时才能替换，这里就不一一列举了。
##### 读取命令
1. get
get命令的key可以表示一个或者多个键，键之间以空格隔开
2. gets
gets命令比普通的get命令多返回了一个数字。这个数字可以检查数据是否发生改变。当key对应的数据改变时，这个多返回的数字也会改变。
3. cas
cas即checked and set的意思
只有当最后一个参数和gets所获取的参数匹配时才能存储，否则返回“EXISTS”。
##### 状态命令
这些命令是我最喜欢用的了。。。因为都可以查看全局资源。
1. stats
<img src ="/img/SSM3.png"/>
2. stats items
<img src ="/img/SSM4.png"/>
执行stats items，可以看到STATitems行，如果memcached存储内容很多，那么这里也会列出很多的STAT items行。
3. stats cachedump slab_id limit_num
<img src ="/img/SSM5.png"/>
limit_num看起来好像是返回多少条记录，猜的一点不错， 不过0表示显示出所有记录，而n(n>0)就表示显示n条记录，如果n超过该slab下的所有记录，则结果和0返回的结果一致。

其他memcached的命令在这里就不一一赘述了，网上都能查到。

### 项目中的SSM实际应用
在项目中使用SSM除了有个memcached的服务器还不够，还需要相关的api，这里贴出我的网盘地址，可以下载参考。[点我下载](http://pan.baidu.com/s/1c09C5zU)提取码为kqfz
解压后将lib文件夹和dist文件夹里的jar文件复制到项目中，记得删除重复的低版本jar文件，不然会报错。
然后在applicationContext.xml文件中添加如下配置：

     <!-- ***********************  simple-spring-memcache ******************************* -->
    <!-- ssm 配置文件，主要用来加载组件核心的Advice，供程序调度使用；封装在 simple-spring-memcached-3.1.0.jar 文件中-->
    <import resource="simplesm-context.xml" />
    <!-- 让代理机制起到作用,simple-spring-memcached主要是基于AOP的代理  -->
    <aop:aspectj-autoproxy />    
    <!-- com.google.code.ssm.CacheFactory是一个FactoryBean，会返回Cache（高速缓存）实体供Advice使用 -->
    <!-- FactoryBean解决的是如何创建无法直接通过new运算符创建的Bean，并注入到其他的bean中。也就是说FactoryBean是创建或者管理其他被注入和管理Bean的工厂Bean -->
    <bean name="defaultMemcachedClient" class="com.google.code.ssm.CacheFactory">
        <property name="cacheClientFactory">
            <!-- xmemcached配置方法 -->
            <bean name="cacheClientFactory" class="com.google.code.ssm.providers.xmemcached.MemcacheClientFactoryImpl" />
            <!--  spymemcached配置方法
            <bean name="cacheClientFactory" class="com.google.code.ssm.providers.spymemcached.MemcacheClientFactoryImpl" />
            -->
        </property>
        <!-- 定义了缓存节点的IP地址和端口号 -->
        <property name="addressProvider">
            <bean class="com.google.code.ssm.config.DefaultAddressProvider">
                <property name="address" value="127.0.0.1:11211" />
            </bean>
        </property>
        <!-- 定义了缓存节点的查找方法 -->
        <property name="configuration">
            <bean class="com.google.code.ssm.providers.CacheConfiguration">
                <property name="consistentHashing" value="true" />
            </bean>
        </property>
    </bean>
    <bean class="com.google.code.ssm.Settings">
    <property name="order" value="500" />
  </bean>
	<!-- ***********************  simple-spring-memcache ******************************* -->    

接下来就是在代码中使用切面来实现你的缓存了yeah~
具体代码可以参考我前面写过的一篇文章。
这里有几点我遇到的问题提一下:
生成key如果对象方法里没有@cachemethod的话,会调用toString()方法。我一般重写toString（）方法来规定key的生成规则。
具体如下：

<img src="/img/SSM6.png"/>
<img src="/img/SSM7.png"/>





