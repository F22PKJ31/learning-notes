# 雪花算法（SnowFlake）Java 实现 - 简书

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/2a27fbd9e71a)
>
> 分布式 id 生成算法的有很多种，Twitter 的 SnowFlake 就是其中经典的一种。

## 算法原理

SnowFlake 算法生成 id 的结果是一个 64bit 大小的整数，它的结构如下图：

![](http://upload-images.jianshu.io/upload_images/13382703-b64e38457ddd13e2.jpg) 结构图

1. **1bit**，不用，因为二进制中最高位是符号位，1 表示负数，0 表示正数。生成的 id 一般都是用整数，所以最高位固定为 0。
2. **41bit - 时间戳**，用来记录时间戳，毫秒级。
   * 41 位可以表示![](https://math.jianshu.com/math?formula=2%5E%7B41%7D-1)个数字，  
   * 如果只用来表示正整数（计算机中正数包含 0），可以表示的数值范围是：0 至 ![](https://math.jianshu.com/math?formula=2%5E%7B41%7D-1)，减 1 是因为可表示的数值范围是从 0 开始算的，而不是 1。  
   * 也就是说 41 位可以表示![](https://math.jianshu.com/math?formula=2%5E%7B41%7D-1)个毫秒的值，转化成单位年则是!\[\]\([https://math.jianshu.com/math?formula=\(2%5E%7B41%7D-1\) %2F \(1000 \* 60 \* 60 \* 24 \*365\) %3D 69\)年](https://math.jianshu.com/math?formula=%282%5E%7B41%7D-1%29%20%2F%20%281000%20*%2060%20*%2060%20*%2024%20*365%29%20%3D%2069%29年)
3. **10bit - 工作机器 id**，用来记录工作机器 id。
   * 可以部署在![](https://math.jianshu.com/math?formula=2%5E%7B10%7D%20%3D%201024)个节点，包括 5 位 datacenterId 和 5 位 workerId  
   * 5 位（bit）可以表示的最大正整数是![](https://math.jianshu.com/math?formula=2%5E%7B5%7D-1%20%3D%2031)，即可以用 0、1、2、3、....31 这 32 个数字，来表示不同的 datecenterId 或 workerId
4. **12bit - 序列号**，序列号，用来记录同毫秒内产生的不同 id。
   * 12 位（bit）可以表示的最大正整数是![](https://math.jianshu.com/math?formula=2%5E%7B12%7D-1%20%3D%204095)，即可以用 0、1、2、3、....4094 这 4095 个数字，来表示同一机器同一时间截（毫秒\) 内产生的 4095 个 ID 序号。

由于在 Java 中 64bit 的整数是 long 类型，所以在 Java 中 SnowFlake 算法生成的 id 就是 long 来存储的。

**SnowFlake 可以保证：**

> 1. 所有生成的 id 按时间趋势递增
> 2. 整个分布式系统内不会产生重复 id（因为有 datacenterId 和 workerId 来做区分）

## 算法实现（Java）

[Twitter 官方给出的算法实现](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Ftwitter-archive%2Fsnowflake%2Fblob%2Fsnowflake-2010%2Fsrc%2Fmain%2Fscala%2Fcom%2Ftwitter%2Fservice%2Fsnowflake%2FIdWorker.scala) 是用 Scala 写的，这里不做分析，可自行查看。

Java 版算法实现：[搬运自煲煲菜的博客](https://links.jianshu.com/go?to=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000011282426)

```text
public class IdWorker{

    //下面两个每个5位，加起来就是10位的工作机器id
    private long workerId;    //工作id
    private long datacenterId;   //数据id
    //12位的序列号
    private long sequence;

    public IdWorker(long workerId, long datacenterId, long sequence){
        // sanity check for workerId
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0",maxWorkerId));
        }
        if (datacenterId > maxDatacenterId || datacenterId < 0) {
            throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0",maxDatacenterId));
        }
        System.out.printf("worker starting. timestamp left shift %d, datacenter id bits %d, worker id bits %d, sequence bits %d, workerid %d",
                timestampLeftShift, datacenterIdBits, workerIdBits, sequenceBits, workerId);

        this.workerId = workerId;
        this.datacenterId = datacenterId;
        this.sequence = sequence;
    }

    //初始时间戳
    private long twepoch = 1288834974657L;

    //长度为5位
    private long workerIdBits = 5L;
    private long datacenterIdBits = 5L;
    //最大值
    private long maxWorkerId = -1L ^ (-1L << workerIdBits);
    private long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);
    //序列号id长度
    private long sequenceBits = 12L;
    //序列号最大值
    private long sequenceMask = -1L ^ (-1L << sequenceBits);

    //工作id需要左移的位数，12位
    private long workerIdShift = sequenceBits;
   //数据id需要左移位数 12+5=17位
    private long datacenterIdShift = sequenceBits + workerIdBits;
    //时间戳需要左移位数 12+5+5=22位
    private long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;

    //上次时间戳，初始值为负数
    private long lastTimestamp = -1L;

    public long getWorkerId(){
        return workerId;
    }

    public long getDatacenterId(){
        return datacenterId;
    }

    public long getTimestamp(){
        return System.currentTimeMillis();
    }

     //下一个ID生成算法
    public synchronized long nextId() {
        long timestamp = timeGen();

        //获取当前时间戳如果小于上次时间戳，则表示时间戳获取出现异常
        if (timestamp < lastTimestamp) {
            System.err.printf("clock is moving backwards.  Rejecting requests until %d.", lastTimestamp);
            throw new RuntimeException(String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds",
                    lastTimestamp - timestamp));
        }

        //获取当前时间戳如果等于上次时间戳（同一毫秒内），则在序列号加一；否则序列号赋值为0，从0开始。
        if (lastTimestamp == timestamp) {
            sequence = (sequence + 1) & sequenceMask;
            if (sequence == 0) {
                timestamp = tilNextMillis(lastTimestamp);
            }
        } else {
            sequence = 0;
        }

        //将上次时间戳值刷新
        lastTimestamp = timestamp;

        /**
          * 返回结果：
          * (timestamp - twepoch) << timestampLeftShift) 表示将时间戳减去初始时间戳，再左移相应位数
          * (datacenterId << datacenterIdShift) 表示将数据id左移相应位数
          * (workerId << workerIdShift) 表示将工作id左移相应位数
          * | 是按位或运算符，例如：x | y，只有当x，y都为0的时候结果才为0，其它情况结果都为1。
          * 因为个部分只有相应位上的值有意义，其它位上都是0，所以将各部分的值进行 | 运算就能得到最终拼接好的id
        */
        return ((timestamp - twepoch) << timestampLeftShift) |
                (datacenterId << datacenterIdShift) |
                (workerId << workerIdShift) |
                sequence;
    }

    //获取时间戳，并与上次时间戳比较
    private long tilNextMillis(long lastTimestamp) {
        long timestamp = timeGen();
        while (timestamp <= lastTimestamp) {
            timestamp = timeGen();
        }
        return timestamp;
    }

    //获取系统时间戳
    private long timeGen(){
        return System.currentTimeMillis();
    }

    //---------------测试---------------
    public static void main(String[] args) {
        IdWorker worker = new IdWorker(1,1,1);
        for (int i = 0; i < 30; i++) {
            System.out.println(worker.nextId());
        }
    }

}
```

算法中大量使用位运算，这里不对位运算做过多解释，代码的详细解释参考[煲煲菜的博客](https://links.jianshu.com/go?to=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000011282426)

> 文章主要摘抄自[煲煲菜的博客](https://links.jianshu.com/go?to=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000011282426)  
> 如有侵权之处请留言告知，会立即删除。

