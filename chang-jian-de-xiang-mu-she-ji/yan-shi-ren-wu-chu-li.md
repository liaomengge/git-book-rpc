一般项目中都会遇到很多延时任务处理，对于一些简单的定时任务只需要简单的Corn处理即可，但是对于复杂的延时规则或者大量的延时处理，那么简答的处理方案将满足不了我们的需求了。之前，在项目中遇到一个场景，将消息推送出去，如果失败；那么将阶梯式的延时处理该任务（比如：10min，30min，1h再处理），故下面几篇主要是介绍一些常见的延时任务处理~

延时任务处理主要分2种：

一、内存型处理

[Min Heap Queue](/chang-jian-de-xiang-mu-she-ji/yan-shi-ren-wu-chu-li/zui-xiao-dui-dui-lie.md)、[Circle Queue](/chang-jian-de-xiang-mu-she-ji/yan-shi-ren-wu-chu-li/huan-xing-dui-5217-shi-jian-lun.md)

优点：快

缺点：数据易丢失，需要对数据进行固化

二、分布式处理

[Loop Scan Db/Redis](/chang-jian-de-xiang-mu-she-ji/yan-shi-ren-wu-chu-li/loop-scan-dbredis.md)、[Redis Sort Set](/chang-jian-de-xiang-mu-she-ji/yan-shi-ren-wu-chu-li/redis-sort-set.md)、[Redis KeySpace Notifications](/chang-jian-de-xiang-mu-she-ji/yan-shi-ren-wu-chu-li/redis-keyspace-notifications.md)、[MQ DLE/DLQ](/chang-jian-de-xiang-mu-she-ji/yan-shi-ren-wu-chu-li/mq-dledlq.md)

优点：分布式，支持大数据量

缺点：处理时间不是很精确，需要额外的资源开销

建议：

如果数据延时比较规则（比如：都延时10min等）或者有特性，且数据量不是非常大时，建议：MQ DLE/DLQ

如果对数据量不大的，Key过期不频繁的，精确度比较高时，建议使用：Redis KeySpace Notifications

如果数据量比较大，延时不规则，精度略高时，建议使用：Redis Sort Set



参考资料

