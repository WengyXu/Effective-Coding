# Effective-Coding
程序猿避免坑指南

* 禁止使用MongoRepository操作mongodb（mysql也一样），所有操作只能使用MongoTemplate。（MongoRepository会自动帮你完成对象转换，复杂对象时会造成内存泄露）

* 日志对象一定要使用private static final修饰
    * 定义成static final,logger变量不可变，读取速度快
    * 一些日志框架（log4j）会在打印日志的时候使用sychronized关键字，加上static以后没个类只会产生一个实例，这样在并发调用这个类的时候就不会因为抢不到Monitor而处于阻塞状态，可以提高系统的吞吐量
    * static 修饰的变量是不管创建了new了多少个实例，也只创建一次，节省空间，如果每次都创建Logger的话比较浪费内存；final修饰表示不可更改，常量
    * 将域定义为static,每个类中只有一个这样的域.而每一个对象对于所有的实例域却都有自己的一份拷贝.，用static修饰既节约空间，效率也好。final 是本 logger 不能再指向其他 Logger 对象

</br></br>
* 对象的复制自己手动用set方法，不允许使用BeanUtils.copyProperties。（一个11个属性的对象通过BeanUtils.copyProperties与set方法耗时相差了500多倍）

* 不允许在for循环中查询数据，如果为redis可以使用mget或者pipline，优先使用mget。如果为mongodb可以使用in查询，另外必须保证in的字段有索引。

* redis的所有赋值操作都必须要带上失效时间，不允许存在永不失效的数据。不允许拿redis当数据库使用。

* 谨慎使用java的反射，如果一个对象在整个应用里面只仅初始化时反射一次，可以使用，如果需要被多次反射，不允许使用。

* mysql查询时不允许直接select * ，需要什么字段返回什么字段。

* redis：禁止使用keys、flushall、flushdb命令。

* redis：大key的删除，不允许直接使用del，Hash删除: hscan + hdel、List删除: ltrim、Set删除: sscan + srem、 SortedSet删除: zscan + zrem，也可以使用redis4.0的unlink（目前大部分sdk并未支持）

* git合并，不能为了方便，分两次合并，中途只合并部分，第二次合并会影响别人分支的代码（因为在合并部分以后，你的代码已经是最新的了，别人的代码已经修改的代码，在二次合并和，会被覆盖）

* 必须使用自定义线程池（设置线程名，最大线程数，重写异常处理方法，丢弃策略）

* 使用ThreadLoacl一定要注意，在使用完后，进行remove,防止内存泄露和串信息

* DeferredResult（只能用于单独处理慢请求，做到线程隔离） 实现异步，需要设置超时时间，多个DeferredResult不能使用相同的连接池

* HashSet 不是线程安全的，换为 ConcurrentHashMap同时把 value 写死一样可以达到 set 的效果。初始化 ConcurrentHashMap 的大小尽量大一些，避免频繁的扩容。

* MySQL 中很多数据都已经不用了，进行冷热处理。尽量降低单表数据量。同时后期考虑分表。
	* Mysql的流水表从一开始就要考虑冷热数据分离的问题
</br></br>

* 线程池的名称一定得取的有意义，不然是自己给自己增加难度。
	* 根据监控将线程池的队列大小调整为一个具体值，并且要有拒绝策略。
	
</br></br>
* 类的静态属性field不能直接赋值，会造成NoClassDefFoundError java.lang.ExceptionInInitializerError 是因为类在加载的时候会将静态属性class init 如果赋值失败就会报这个错误

* 使用微服务远程调用（ribbon,fegin），会隐藏远程host ip,要注意打印ip,否则没法排查工作。

* 项目必须配置3个数据源 读，写，和主库读，主库读用来操作那些对实时性要求很高的数据

* 注册中心注册后，一定要检查注册进去的IP地址，防止出现多网卡使用了错误的IP

大家可以在下面多评论，多给大家来点良药
