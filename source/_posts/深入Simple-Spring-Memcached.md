title: 初学Simple-Spring-Memcached
date: 2015-08-17 16:13:41
categories: 技术
tags: [SSM,缓存]
---
Simple-Spring-Memcached（简称ssm），它也是一个通过Annatation与AOP来完成缓存数据操作的开源项目。仔细看了一下代码，基本上把我之前碰到的问题都解决了，而且MultiCache这一块的实现超出我的预期。该项目主要优点如下：
 
与Spring完善集成
支持两种Memcached Java Client (spymemcached,Xmemcached)
基于Annotation方式实现缓存操作，对代码侵入性小
annotation丰富，可以满足绝大部分需求
下面介绍一下其中各annotation的使用。ssm项目中的Annotation主要分成以下几类
 
SingleCache类 操作单个POJO的Cache数据，由ParameterValueKeyProvider和CacheKeyMethod来标识组装key
MultiCache类 操作List型的Cache数据，由ParameterValueKeyProvider和CacheKeyMethod来标识组装key
AssignCache类 指定key操作Cache数据，由annotation中的 assignedKey 指定key
 
 
simple-spring-memcached本质上是采用了AOP的方式来实现缓存的调用和管理，其核心组件声明了一些Advice，当遇到相应的切入点时，会执行这些Advice来对memcached加以管理。
 
 
 
切入点是通过标签的方式来进行声明的，在项目开发时，通常在DAO的方法上加以相应的标签描述，来表示组件对该方法的拦截
组件所提供的切入点主要包括以下几种：
ReadThroughSingleCache、ReadThroughMultiCache、ReadThroughAssignCache
当遇到查询方法声明这些切入点时，组件首先会从缓存中读取数据，取到数据则跳过查询方法，直接返回。
取不到数据在执行查询方法，并将查询结果放入缓存，以便下一次获取。

InvalidateSingleCache、InvalidateMultiCache、InvalidateAssignCache
当遇到删除方法声明这些切入点时，组件会删除缓存中的对应实体，以便下次从缓存中读取出的数据状态是最新的.

UpdateSingleCache、UpdateMultiCache、UpdateAssignCache

###各Annotation的详细说明
 
ReadThroughSingleCache
作用：读取Cache中数据，如果不存在，则将读取的数据存入Cache
key生成规则：ParameterValueKeyProvider指定的参数，如果该参数对象中包含CacheKeyMethod注解的方法，则调用其方法，否则调用toString方法
 
    @ReadThroughSingleCache(namespace = "Alpha", expiration = 30)
	   public String getDateString(@ParameterValueKeyProvider final String key) {
	       final Date now = new Date();
	       try {
	           Thread.sleep(1500);
	       } catch (InterruptedException ex) {
	       }
	       return now.toString() + ":" + now.getTime();
	   }    
　　
 
InvalidateSingleCache
作用：失效Cache中的数据
key生成规则：
 
 
使用 ParameterValueKeyProvider注解时，与ReadThroughSingleCache一致
使用 ReturnValueKeyProvider 注解时，key为返回的对象的CacheKeyMethod或toString方法生成
 
 
    @InvalidateSingleCache(namespace = "Charlie")
	    public void updateRandomString(@ParameterValueKeyProvider final Long key) {
	        // Nothing really to do here.
	    }
	 
	    @InvalidateSingleCache(namespace = "Charlie")
	    @ReturnValueKeyProvider
	    public Long updateRandomStringAgain(final Long key) {
	        return key;
	    }    


UpdateSingleCache
作用：更新Cache中的数据
key生成规则：ParameterValueKeyProvider指定
ParameterDataUpdateContent：方法参数中的数据，作为更新缓存的数据
ReturnDataUpdateContent：方法调用后生成的数据，作为更新缓存的数据
注：上述两个注解，必须与Update*系列的注解一起使用
 
 
    @UpdateSingleCache(namespace = "Alpha", expiration = 30)
	   public void overrideDateString(final int trash, @ParameterValueKeyProvider final String key,
	           @ParameterDataUpdateContent final String overrideData) {
	   }
	 
	   @UpdateSingleCache(namespace = "Bravo", expiration = 300)
	   @ReturnDataUpdateContent
	   public String updateTimestampValue(@ParameterValueKeyProvider final Long key) {
	       try {
	           Thread.sleep(100);
	       } catch (InterruptedException ex) {
	       }
	       final Long now = new Date().getTime();
	       final String result = now.toString() + "-U-" + key.toString();
	       return result;
	   }    


ReadThroughAssignCache
作用：读取Cache中数据，如果不存在，则将读取的数据存入Cache
key生成规则： ReadThroughAssignCache 注解中的 assignedKey 字段指定
 
 
    @ReadThroughAssignCache(assignedKey = "SomePhatKey", namespace = "Echo", expiration = 3000)
	    public List<String> getAssignStrings() {
	        try {
	            Thread.sleep(500);
	        } catch (InterruptedException ex) {
	        }
	        final List<String> results = new ArrayList<String>();
	        final long extra = System.currentTimeMillis() % 20;
	        final String base = System.currentTimeMillis() + "";
	        for (int ix = 0; ix < 20 + extra; ix++) {
	            results.add(ix + "-" + base);
	        }
	        return results;
	    }    


InvalidateAssignCache
作用：失效缓存中指定key的数据
key生成规则：assignedKey 字段指定
 
 
	@InvalidateAssignCache(assignedKey = "SomePhatKey", namespace = "Echo")
    public void invalidateAssignStrings() {
    }	


UpdateAssignCache
作用：更新指定缓存
key生成规则：assignedKey 字段指定
 
 
	@UpdateAssignCache(assignedKey = "SomePhatKey", namespace = "Echo", expiration = 3000)
    public void updateAssignStrings(int bubpkus, @ParameterDataUpdateContent final List<String> newData) {
    }	