时间轮\(HashedWheelTimer\)其实很简单，就是一个循环队列，如下图所示，

![](http://upload-images.jianshu.io/upload_images/584578-044ce81079679c1c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

原始时间轮：如下图一个轮子，有8个“槽”，可以代表未来的一个时间。如果以秒为单位，中间的指针每隔一秒钟转动到新的“槽”上面，就好像手表一样。如果当前指针指在1上面，我有一个任务需要4秒以后执行，那么这个执行的线程回调或者消息将会被放在5上。那如果需要在20秒之后执行怎么办，由于这个环形结构槽数只到8，如果要20秒，指针需要多转2圈。位置是在2圈之后的5上面（20 % 8 + 1）。这个圈数需要记录在槽中的数据结构里面。这个数据结构最重要的是两个指针，一个是触发任务的函数指针，另外一个是触发的总第几圈数。时间轮可以用简单的数组或者是环形链表来实现。

怎么实现时间轮呢？Netty中已经有了一个时间轮的实现，[HashedWheelTimer.java](https://github.com/netty/netty/blob/4.1/common/src/main/java/io/netty/util/HashedWheelTimer.java)，可以参考它的源代码。



