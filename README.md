内存泄漏、性能优化：
1、内存泄漏
原因：
a、生命周期短的变量被生命周期长短变量持有，导致该变量不在使用的时候无法回收
b、Handler
c、线程
d、非静态内部类
e、单例
排查内存泄漏的工具：
a、android memory monitor观察重复进入一个页面，使用内存是否持续增加
b、dump 进入一个页面前后的Java heap
c、使用Mat 比较两个heap 文件中的对象数，通过正则表达式过滤
d、针对NDK部分，需要观察systrace 数据，判定是否JNI中有内存泄漏。
e、Debug、Runtime中都有接口可以输出当前内存使用情况。

2、性能优化
a、枚举占用更多内存，但是如果需要强调类型安全，还是得用
b、自动装箱，可以考虑使用SparseArray 替换 HashMap，HashMap相比更加耗费内存
c、避免Over draw，如果和主题颜色一致，不需要在设置背景色
d、invalidate() 用invalidateInternal（l,t,r,b,）替换
e、BaseAdapter 中 inflater的时候传入parent，和false
f、减少UI层级，尝试用相对布局代替线性布局；考虑使用merge、include、viewstub
g、动态加载UI
h、alpha值慎用，TextView可以通过设置颜色，替代setAlpha
i、使用线程池替代频繁创建线程
j、使用synchronized 代码块 替代 synchronized 方法；减小同步区，如单例；
k、尽量不要在循环、draw等频繁调用的方法活着代码段中创建对象；
l、字符串拼接使用StringBuilder，而不是StringBuffer
m、图片、数据采用多级缓存
n、图片、数据采用异步加载
o、控制动画的刷新频率来提高性能
性能优化相关工具：
a、TraceView：能够看到每个方法调用的次数、事件等
b、GPU profiling：能看到每帧花费的时间，以及绘制的不同阶段占用的时间比例
c、Overdraw（Debug GPU overdraw 选项）：颜色越深，重绘制的越多次
d、Lint：可以查出部分Overdraw的UI、以及布局不合理的UI


Android：
1、Activity、Window 、View
2、Fragment、Activity的区别、优点
3、Window的生命周期、类型
 系统窗口
 应用窗口
 子窗口
 在Activity创建之后的attach方法中和Activity绑定
 在Activity performDestroy 方法中 调用Window destroy方法

 4、自定义控件相关，事件截获、Measure、layout、draw等
 5、两种动画的实现方式：View动画，和属性动画
 View动画：
 在View startAnimation的时候调用invalidate触发绘制
 通过父控件调整子控件Canvas的坐标系来实现
 drawDispatch-drawChild-draw-drawAnimation
 每次绘制完成之后判断是否动画结束，没有结束重新调用invalidate触发重绘制。
 View动画绘制的频率由View控制，很难修改
 invalidate只是标记为无效，当收到VSYNC型号之后，系统就会重新绘制，VSYNC频率60HZ
 属性动画：
 它将AnimationHandler 设置到了Choreographer单例中，当其收到VSYNC信号后，会回调AnimationHandler的run方法，然后去修改对应对象的属性值（采用反射的方式）。
 Choreographer为线程局部变量，由ThreadLocal，每一个线程对应一个实例。


 6、Android Binder机制
 7、AMS、WMS的机制
 8、DP、pixel、物理尺寸、density的关系
 density = DPI值 ／ 160 ＝ pixel / dp
 1 dp = density pixel = (DPI / 160 )* 1 pixel 
 若DPI ＝ n；
  1 dp ＝ (n/160) * 1pixel
  1 pixel = 1/n inch
  即：
  1dp = 1/160 inch


  9、4中launchMode、以及它们的使用场景
  a、standard
  创建一个新的Activity
  b、singleTop
  栈顶不是该Activity，创建一个新的Activity。否则，onNewIntent
  适合接受通知启动的内容页面。
  例如：新闻客户端的新闻内容页面，如果收到10个新闻推送，每次打开一个内容页很烦。
  其他推送也是一样。
  我们的项目中私聊页面也是一样的，点击通知之后打开一个私聊页面，再次点击，只是刷新数据；
  其他消息页面也是相同的处理
  c、singleTask
  栈中没有该Activity，创建一个新的Activity。否则，onNewIntent＋clearTop
  适合作为程序的入口点。
  例如浏览器的主页面；
  我们App的MainActivity
  微信、支付宝的支付页面也是一个入口点
  第三方的分享页面
  d、singleInstance
  栈中只有这一个Activity，没有其他
  适合需要与程序分离的页面。
  例如闹铃提醒，需要和闹铃设置分离。（singleInstance不适合用于中间页面）

  
  10、androd碎片化、版本差异

  Java 部分：
  1、Java的4种缓存模式，各自的特点
  强引用：不会被GC回收
  软引用：当内存不够的时候被GC回收
  弱引用：无论内存够不够，GC的时候都会回收
  虚引用：不常用
  注意：因为弱引用的GC回收特性用于缓存的时候更加倾向于软引用；

  2、线程池的几种使用方式、核心线程数、扩展线程数，以及使用场景
  newCachedThreadPool
  创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
  newFixedThreadPool 
  创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。可以根据系统资源进行配置，例如Runtime.getRuntime().availableProcessors();
  newScheduledThreadPool 
  创建一个定长线程池，支持定时及周期性任务执行。
  newSingleThreadExecutor 
  创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。适用于读写数据库；

  3、GC部分
  A、可达性分析，GCRoots种类：
  虚拟机栈（栈帧中的本地变量表）中引用的对象
  方法区中类静态属性引用的对象
  方法区中常量引用的对象
  本地方法栈中JNI（Native方法）引用的对象。
  B、收集算法：
  标记清除：标记不可达的对象，然后回收；缺点是效率不高，还产生内存碎片
  复制算法：将内存分为大小相同的两块，将可达的对象复制到空闲的内存上；效率高，内存利用率低； 优化算法是将内存新生代划分为2*survivor、1*Eden，每次使用1*survivor、1*Eden，回收时将可达对象复制到另一个survivor；survivor：eden ＝ 1:8；
  标记整理算法：通常用于分代算法中老年代的收集；先标记对象是否可达，然后将存活对象向内存一侧移动，清理掉边界以外的内存；
  C、垃圾收集器：
  新生代：Serial（单线程）、ParNew、ParallelScavenge（侧重吞吐量）、G1
  年老代：CMS、Serial Old（MSC）、Parallel Old 


  4、Java内存模型，Java Memory Model，JMM
  为了解决不同线程对共享对象的访问，即多线程同步问题。
  Java 线程－－－－> 工作内存－－－－> Save｜Load－－－> 主内存
  定义了八种操作：
  lock：锁定
  unlock：解除锁定
  read:从主内存读到工作内存
  load:将上述读到的变量放入变量副本中
  use:将工作内存中变量传递给执行引擎，使用
  assign:给工作内存中的变量赋值，（来至于执行引擎）
  store:将工作内存的变量传送给主内存
  write:将上述变量放入主内存的变量中。
  3个特性：
  原子性：通过lock、unlock，更高层次的字节码monitorenter、monitorexit，以及synchronized来保证原子性。
  可见性：通过修改后同步回主内存，读取前从主内存刷新变量值的方式实现。volatile final synchronized 都可以
  有序性：如果在线程内观察，所有的操作都是有序的；如果在一个线程中观察另一个线程，所有操作都是无序的；可以通过volatile , synchronized保证

  5、线程组、守护线程
  守护线程：在所有用户线程结束之后，JVM结束；使用时需要在start之前调用setDaemon(true)；例如，垃圾回收线程为守护线程。
  线程组：用于对线程进行管理，例如调用ThreadGroup的interrupt方法，回对其中所有的线程进行中断。包括其子线程组中的线程。


  6、Java中集中基本数据结构、排序算法
  ArrayList 数组列表, 非同步，可以为空
  LinkedList 双向链表，非同步，可以为空
  HashMap 非同步，key、value可以为null；; 采用数组＋链表的数据结构，每一个数组元素都是一个链表；用hash & size－1得到数组的索引，找到在数组中的位置，然后将健值对放在链表的头部；采用链地址法来解决hash冲突；问题是每次扩容都＊2，容易导致内存浪费；另外在数据比较大多时候，遍历链表比较慢；还有它会对int等基础类型自动装箱，即int－》integer，浪费性能；
  HashTable：  同步（synchronized），key、value都不能为null
  	
	Set
	ArrayMap：key可以不为int，思路和SparseArray类似，将key的hash-code存放在数组中，然后排序，索引值＊2为kay在数组中的索引。它的一个整形数组存放key的hatched值，一个数组存放kay－value键值对。
	SparseArray：适合key为int的情况，扩容的时候也是＊2；采用两个数组，一个整形数组存储key，另外一个存value；整形数组需要排好序，对应的索引也为value的索引。查找通过二分查找。

	7、Java异常种类，Android OOM的处理
	异常三种用法：
	throws InterruptException 在方法尾部调用
	throw new RunnableException() 在方法体中调用
	try catch 捕获异常
	Throwable的两个子类Error 和 Exception ，其中：
	Error 表示错误，无法恢复，不应该被捕获，主要有：
	VirtualMachineError 虚拟机错误
	OutOfMemoryError 内存溢出
	StackOverflowError 栈溢出
	LinkageError 链接错误
	Exception 分为两种类型 Checked 和 Runtime
	Checked：
	Java程序必须处理这种类型的异常，否则无法编译通过，两种处理方式：
	a、知道怎么处理该异常，使用try catch捕获异常，并且处理
	b、不知道怎么处理异常，在方法尾部 throws 抛出该异常，让上层处理。
	典型的有：
	反射的时候 ClassNotFoundException, NoSuchMethodException
	文件等IO操作的时候 IOException
	线程sleep的时候 InterruptException
	底层Api抛出的这些异常，在上层代码中都必须要捕获，给予处理。
	注意：CheckedException 为接口
	RuntimeException:
	不要求程序显示的捕获这种类型的异常，但是可以加上; 通常会交给缺省的异常处理程序。
	常见的有：
	ClassCastException
	IndexOutOfBoundsException
	NullPointException
	RuntimeException 为实体类
	NoSuchSymbolException 找不到符号对应的类
	ArithmeticException 算数异常，例如除以0


	8、线程安全、线程同步、同步锁、ReentrantLock
	线程安全的5类：
	a、不可变：例如final 修饰的基础类型，String等
	b、绝对线程安全：
	c、相对线程安全：Vector、HashTable等
	d、线程兼容：例如HashMap、ArrayList等
	e、线程对立：Thread 等 suspend 和 resume方法，如果两个线程同时尝试暂停和恢复一个线程，很可能发生死锁。
	线程安全的3中方式：
	a、互斥同步：（阻塞、悲观）
	最基本的就是synchronized，会在代码块的前后生成两个字节码monitorEnter, monitorexit
	同时关联一个对象引用；对同一线程时可重入的，例如单例的双重判定；
	另外一种方式为ReentrantLock:
	等待可中断：awaitUntil等待持续多长时间，可以被唤醒，类似wait(long time)
	可实现公平锁：可设置为公平锁，这样会按照请求锁的顺序获取锁；synchronized 非公平的
	锁可以绑定多个条件：ReentrantLock可以创建多个Condition，分别用来实现await 和signal
	类似于synchronized()绑定不同的对象。
	用法：
	在try中调用lock方法，在finally中调用unlock方法；Condition 的await方法必须喝lock配合使用，这点和synchronized类似；signal 类似于 notify，用于唤醒 await等待。
	b、非阻塞同步：（乐观）
	例子：AtomicInteger的自增操作是原子性的，多线程同时对它做自增的时候也能保证同步。
	c、无同步方案：
	纯代码、可重入代码，没有共享变量活着共享变量为final都是线程安全的。例如使用ThraadLocal 存储的变量
	另外一些虚拟机优化措施：
	自旋锁，自适应自旋锁：本来要挂起等待的线程暂时原地等待几个周期
	锁消除：监测到不存在共享数据，去掉锁
	粗化锁：将循环里面的锁移到循环外面，避免反复加锁
	轻量级锁：
	偏向锁：偏向第一个获取它的线程。甚至消除同步。

	9、网络请求，使用签名算法；https，以及证书、证书链
	网络请求两种方式：
	HttpClient：
	HttpUrlConnection：
	两种框架：
	volley：
	OKhttp：
	签名算法：
	使用MD5对content生成数字指纹（摘要），然后用私钥对指纹进行加密。以此来保证数据不被修改，或者识别是否被修改。
	另外用Base64对content进行加密，避免传递明文。
	Base64: 从字符串获取字节数组，每6bit映射为一个字符（可定义）；可生成密文，缺点是字节数增加1/3
	证书：
	包含：颁发机构信息、使用机构信息、公钥、加密算法、指纹签名算法、指纹、指纹提取算法等
	使用机构自己生成一对公钥、私钥；然后将公钥以及相关信息发送给颁发机构，颁发机构生成一个证书，然后采用指定算法提取指纹，并用自己的私钥对指纹签名（加密）；这个时候证书一旦被第三方修改，因为不知道颁发机构的私钥，无法重新完成签名。
	客户的拿到服务器提供的证书之后，会在本地查找其颁发机构证书并且提取公钥，用颁发机构的公钥给服务器提供的证书中等指纹解密，和重新生成的服务器证书指纹比较是否被修改，没有被修改，就信任该证书。
	证书链：
	服务器提供的证书需要其颁发机构给予签名、认证；其颁发机构的证书可能同样需要更高级别的证书来签名、认证；
	通常根证书又微软、google、苹果等开发操作系统的公司添加到系统里面的。
	单同时也自称用户自己添加信任的证书，比如12306、阿里巴巴
	https 3次握手：
	建立连接
	传递证书给客户端
	确定对称加密算法以及相应密钥


	10、Object中的方法
	构造函数
	clone
	internalClone
	hashcode
	为了保证每个对象的hashcode唯一，算法有很多中，例如：String的hashcode 算法：
	hash ＝ hash ＊31 ＋ chas［i］；
	还有通过静态变量加上一个常量的；
	equals
	通常可以根据hashcode是否一样来判断；例如String，先判断是否为String，然后判断长度是否相等，再次判断hashcode是否相同，再次依次比较每个字符串。
	wait
	notify
	finalize
	对象被回收的时候调用，通常会在finalize线程中调用，并且最多只调用一次


	11、wait、sleep、join、yield、suspend区别
	wait: 属于Object，可能抛出中断异常，执行wait后锁被自动释放，需要和synchronized配合使用；对应的notify 也需要和synchronized配合使用，并且调用notify之后不释放锁。
	sleep：属于Thread，可能抛出中断异常，静态方法；执行sleep 之后不会立即释放锁。不必须喝synchronized配合使用；sleep必须设置睡眠时间；
	suspend：暂停一个线程，如果在同步锁内，它并且不会释放锁，而且除非呗resume，否则不会被唤醒。另外一个问题是不同步，和stop一样，在数据修改到一半的时候suspend，也会导致数据不同步到问题。当然这个问题换成使用sleep也会发生。
	yield：表示本线程放弃CPU资源，让给其他线程，但是时间不确定，可能马上又获取。
	join：Thread，非静态方法本质是通过wait来中断当前线程，所以也会释放锁，如果当前线程被中断，抛出异常


	12、volatile
	可见性：保证被修饰打变量对所有线程可见，通过每次使用都回去主内存读来实现
	禁止指令重排序优化：例如线程A初始化类C，然后标记已经初始化；线程B判断是非已经初始化然后使用类C；如果初始化标记被提前设置，线程B可能会出错。
	不具有原子性：原因是对变量的修改本身不是原子的，多个线程可能读取到修改过程中的不同时刻，得到的值也不同。

	13、停止线程，三种方法
	a、设置退出标记，线程读到这个标记之后正常退出。
	b、使用Thread的stop方法，这个方法可能导致不可预测的后果，不推荐使用
	问题1: 强制退出，清理工作不能完成。
	问题2:强制退出，可能在数据只修改了一半点时候退出，导致数据错乱，不同步。
	c、使用interrupt（本地方法）方法，
	interrupted：静态方法；判断当前线程是否已经中断，true表示中断；调用之后会重置状态为false
	isInterrupted:非静态方法；判断线程自身是否已经中断，不重置状态；
	这个方法就是在run方法的循环中（如果有）调用isInterrupted判断当前线程是否被中断，如果是可以通过调用break、return、抛出中断异常等方法结束线程。
	如果线程处于sleep状态，调用interrupt，sleep方法会抛出interruptException，并且重置中断状态。
	如果线程调用interrupt，然后sleep，sleep方法也会抛出中断异常。这里比较好理解，查看sleep方法体可以看到，它会判断是否有中断，若是，抛出中断异常。

	14、ThreadLocal
	通常用于创建线程局部变量，以自身对象作为Key值，将要保存到对象，例如Looper等存放在当前线程Thread的ThreadLocal.ThreadLocalMap中，保证每个线程仅有一个；
	如果通过get获取Value的时候没有对应的对象值，会去initialValue初始化一个，然后保存在Map中，并且这个初始化方法常被重写，用于创建线程本地变量

	15、注解：

	数据库部分：
	1、数据库的特性

	业务部分：
	1、渠道包，通过MateInf中的文件名称来标记渠道号
	2、图片模糊算法，以及优化
	3、模拟器识别的几种策略
	BUILD中的相关属性，IMEI等；短信数、相册照片数；电量、网络信号、传感器等；

	3、直播推流策略、MediaCodec 以及ffmpeg
	4、Service保活的几种策略，各自的适用范围
	a、提高Service到优先级
	b、监听系统广播、第三方应用广播，重新拉活
	c、启动定时器，定时拉活
	d、使用JobSchedule 拉活
	e、启动两个Service进程互相拉活
	f、启动native进程拉活service

	5、地理位置获取策略
	基站、GPS

	6、Mqtt协议结构、字段等
	Mqtt也是应用层协议；优点是协议头部小，（最小）只有2字节；
	Message type 4 bit:
	PINGREQ：type ＝ 12的时候，ping一个心跳包，只需要发送2个字节
	PINGRESP：type ＝ 13 ，服务器响应心跳包，2个字节
	DISConnect： type＝14，客户的发送断开连接命令，2个字节
	心跳时间：0－18小时

	7、几种JSON解析方式、以及优缺点
	Gson 更新 性能更高
	FastJson 停止更新 性能在有些条件下更高（适用范围窄）

	8、XML的几种解析方式、以及Android中XML的解析方式Pull的优点
	DOM：使用广泛，实现W3C标准；操作简单；需要将整个文件读入内存，不适合用来解析大文件。基于文档；修改方便、检索快捷
	SAX：不需要读入整个XML文件，API接口复杂，适合解析大型文件，部分解析；基于事件；无法修改XML树；性能高；适合多次解析的场合
	PULL：基于事件；不需要完全读入；API较SAX简单；同样无法修改XML；同样效率高；适合单次解析的场合



	9、随遇业务架构，模块区分
	登陆－注册
	直播列表
	直播间
	直播录制：
	状态控制、功能添加等

	Profile
	设置页
	消息页
	10、混淆
	4个功能：
	压缩（shrinker）：移除没有用到的类，变量，方法，属性
	优化（optimizer）：非入口类会加上private／static／final，没有用到的参数会被删除，一些方法会变为内联代码；
	obfuscator（混淆）：使用短、没有意义的名字重命名非入口类的类名，变量名，方法名。入口类的名字保持不变；
	预校验（preverifier）：校验代码是否符合java1.6规范。

	11、gradle
	12、热编译
	13、openGL
	openGL的主程序运行在CPU上
	glsl语言（着色器语言）运行在GPU上
	GPU上运行的线程数可以达到百万级别甚至更高；
	每个线程对应每个顶点、图元、片段的处理过程。
	OpenGL是一个状态机，管线就是绘制流程顶点、顶点着色器、图元装配、光栅化、片元着色器、逐片元执行。

	14、网络相关OKHttp、图片相关等ImageLoader
	15、字符编码
	GBK、GB2312、UTF－8、unicode

	设计模式：
	1、六项基本原则
	a、单一职责原则：
	b、开闭原则：对修改封闭、对扩展开放
	c、里氏替换原则：所有引用基类的地方都能透明的使用其子类替换
	d、依赖倒置原则：高层模块不应该依赖底层模块，两者都依赖抽象；抽象不依赖细节；细节依赖抽象；
	模块间的依赖通过抽象发生，实现类之间不发生直接的依赖关系，依赖通过接口或者抽象类产生。
	e、接口隔离原则：将接口小型化，让客户端依赖的接口尽量小；目的是解偶，从而容易重构
	f、迪米特原则：最少知识原则；即一个类应该对耦合的（依赖的）另一个类了解的越少越好

	2、观察者模式及相关使用案例
	Subject <>－－－－ Observer ，一个主题多个观察者；
	案例：登陆事件，多个模块都会注册一个观察者或者说是监听。
	Android 中案例：BaseAdapter，就是采用观察者模式；当ListView setAdapter的时候，会给
	BaseAdapter 设置一个AdapterDataSetObserver，当数据变化我们调用notifyDataSetChanged的时候，会调用 AdapterDataSetObserver onChanged方法，因为它是ListView 父类的内部类，会通知父类去重新requestLayout

	3、单例模式、使用案例以及多种实现方式和各自的区别
	a、懒汉模式：直接给获取单例的静态方法加上synchronized修饰；缺点是每次都同步，开销大，优点是延时初始化，简单；
	b、双重判定（DCL）：只有为空的时候才加上同步 然后去初始化；静态变量需要用volatile修饰
	c、静态内部类单例模式：在静态内部类中初始化，不需要用到同步字段
	d、枚举单例：可以解决反序列化的时候重新创建实例的问题。
	e、容器单例：再程序启动的时候，将单例类型注入统一的管理类的容器中；android中Service就是这样管理的。

	4、另外列举多种设计模式在android中的应用
	装饰模式：
	InputStream
	ImageLoader里面的缓存模块BitmapCache
	Android中Context、ContextImpl 也是使用的装饰模式。

	解释器模式：
	PackageManagerService：解析xml文件部分

	策略模式：动画的差值器部分



