
Bee
=========
世界上有一样的电话号码吗?	

## 分布式ID还存在的问题：  
分段模式与雪花到底有什么区别？	
一个是依赖DB,一个是依赖时间的.		

依赖DB分段批量获取的算法，是可以产生全局唯一，且批内连续单调递增的ID。但多个请求分别调用生成一批,多个批都插入数据到库,还是不会连续的。强依赖DB。	
雪花生成的是不连续，全局唯一,但只能是趋势递增的ID,部分区间连续，整体不连续。过于依赖时间。	

能不能找到一种，既不依赖DB，也不依赖时间的算法呢？答案是，肯定有的，找吧（本文就能找到）！	
DB表自增ID真的不能用在分布式场景吗？	
这个都被大家忽略了:	
DB表自增ID，也是可以改为具有分布式特性的，SerialUniqueId就是！	

## distributed-id
distributed-id 是ORM框架Bee使用的分布式ID生成方案。	
提供不用依赖DB,不用依赖时钟的生成算法;	
提供DB表主键自增也可具有分布式全局唯一数字id特性的生成算法.	
分布式环境下生成连续单调递增(在一个workerid内),全局唯一数字id.	

**Bee** 网址:  
https://github.com/automvc/distributed-id  
**Bee在gitee** 的网址:  
https://gitee.com/automvc/distributed-id

## 环境要求  
jdk1.7+

## 功能介绍: 

**雪花算法：PearFlowerId**  
改进的雪花算法——姑且称为梨花算法吧  （忽如一夜春风来，千树万树梨花开）。	
改进目标：解决雪花算法的时钟回拨问题；部分避免机器id重复时，号码冲突问题。	

**SerialUniqueId**  

SerialUniqueId:分布式环境下生成连续单调递增(在一个workerid内),全局唯一数字id.		
连续单调递增ID生成算法SerialUniqueId：不依赖于时间，也不依赖于任何第三方组件，只是启动时，用一个时间作为第一个ID设置的种子，设置了初值ID后，就可获取并递增ID。在一台DB内与传统的一样，连续单调递增（而不只是趋势递增），而代表DB的workerid作为DB的区别放在高位，从所有DB节点看，则满足分布式DB生成全局唯一ID。本地（C8 I7 16g）1981ms可生成1亿个ID号,利用上批获取，分隔业务，每秒生成过10亿个ID号不成问题，能满足双11的峰值要求。可用作分布式DB内置生成64位long型ID自增主键。只要按本算法设置了记录的ID初值，然后默认让数据库表id主键自增就可以（如MYSQL）。	

**OneTimeSnowflakeId**  
 * OneTimeSnowflakeId，进一步改进了梨花算法。	
 * 不依赖时间的梨花算法，Workerid应放在序号sequence的上一段，且应用SerialUniqueId算法，使ID不依赖于时间自动递增。	
 * 使用不依赖时间的梨花算法OneTimeSnowflakeId，应保证各节点大概均衡轮流出号，这样入库的ID会比较有序，因此每个段号内的序列号不能太多。	
 * 支持批获取ID号。可以一次取一批ID（即一个范围内的ID一次就可以获取了）。可以代替依赖DB的号段模式。	
 * 应用订单号等有安全要求的场景,可随机不定时获取一些批的号码不用即可。	
 * 
 * 可间隔时间缓存时间值到文件，供重启时设置回初值用。若不缓存，则重启时，又用目前的时间点，重设开始的ID。只要平均不超过419w/s，重启时造成的时钟回拨都不会有影响（但却浪费了辛苦攒下的每秒没用完的ID号）。要是很多时间都超过419w/s，证明你有足够的能力做这件事：间隔时间缓存时间值到文件。	
 * 
 * +------+----------------------+----------+-----------+-----------+
 * | sign |     time(second)     | segment  | workerid  | sequence  |
 * +------+----------------------+----------+-----------+-----------+
 *   1 bit        31 bits           9 bits     10 bits     13 bits


Quick start:
=========	
## 1. How to use distributed-id  
#### 1.1 if it is a maven project,add the following dependency  
Notice:now v 1.7.3 still donot commit to maven,you can download the resource code  directly.  
```xml
		<dependency>
			<groupId>org.teasoft</groupId>
			<artifactId>bee</artifactId>
			<version>1.7.3</version>
		</dependency>

		<dependency>
			<groupId>org.teasoft</groupId>
			<artifactId>honey</artifactId>
			<version>1.7.3</version>
		</dependency>
```

#### 1.2  Of course, can download the jar file directly  
	
## 2. example/test case  
   [link](../../../bee-exam/tree/master/src/main/java/org/teasoft/exam/bee/distribution).  
       
  
Contact & Welcome:
=========	
#### Author's email:    honeysoft@126.com  
#### If you have any problem on bee, please let me know kindly! Thank you, so much!  
#### ORM QQ Group: 992650213     WeChat:AiTeaSoft  
#### At the same time, welcome you to join Bee team create a better future. 
