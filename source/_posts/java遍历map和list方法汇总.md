title: java遍历map、set、list方法汇总
date: 2015-08-10 14:04:40
categories: 技术
tags: [遍历,java]
---
##遍历map
###获得键值对
    for(Map.Entry<String, String> entry:map.entrySet()){   
     System.out.println(entry.getKey()+"--->"+entry.getValue());   
	}       最常规的一种遍历方法.
###用迭代
    Set set = map.entrySet();        
	Iterator i = set.iterator();        
	while(i.hasNext()){     
     Map.Entry<String, String> entry1=(Map.Entry<String, String>)i.next();   
     System.out.println(entry1.getKey()+"=="+entry1.getValue());   
	}       
###最复杂的效率低的但是很暴力很灵活的
        public static void workByEntry(Map<String, Student> map) {
        Set<Map.Entry<String, Student>> set = map.entrySet();
        for (Iterator<Map.Entry<String, Student>> it = set.iterator(); it.hasNext();) {
            Map.Entry<String, Student> entry = (Map.Entry<String, Student>) it.next();
            System.out.println(entry.getKey() + "--->" + entry.getValue());
        }
    }
}    

##遍历list
###顺序遍历
	 for (int i = 0; i < list.size(); i++) {
      System.out.println(list.get(i));
    }	
    内部不锁定,效率最高,但是当写多线程时要考虑并发操作的问题。
###iterator遍历
	 Iterator<String> it = list.iterator();
    while (it.hasNext()) {
      System.out.println(it.next());
    }
    这种方式在循环执行过程中会进行数据锁定,性能稍差,同时,如果你想在循环过程中去掉某个元素,只能调用it.remove方法,不能使用list.remove方法,否则一定出并发访问的错误.
###foreach遍历
	for (String s2 : list) {
      System.out.println(s2);
    }
    换汤不换药，不建议使用。

##遍历set
###iterator遍历
	Iterator<String> it1 = set.iterator();
    for (String ss : set) {
      System.out.println(ss);
    }
###foreach遍历
	for (String sss : set) {
      System.out.println(sss);
    }
