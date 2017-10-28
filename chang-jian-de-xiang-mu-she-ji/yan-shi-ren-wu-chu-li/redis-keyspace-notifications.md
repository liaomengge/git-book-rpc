利用Redis Key过期，Pub/Sub Expire事件通知处理 ~

设计：

1、监听指定的Channel通道【\_\_keyevent@{db}\_\_:expired】即可

高可用：

配合Redis Sentinel处理

优点：

利用Redis Key Expire时间处理延时消息，支持大数据量；

缺点：

Redis Key的3种过期策略\[过期策略\]\([http://www.cnblogs.com/chenpingzhao/p/5022467.html?utm\_source=tuicool&utm\_medium=referral ](https://www.gitbook.com/book/liaomengge1/rpc/edit#/edit/master/chang-jian-de-xiang-mu-she-ji/yan-shi-ren-wu-chu-li/mq-dledlq.md?_k=zo8z3i "abc")\)如果，此时有大量数据量的过期，那么延时任务可能不精确，当然，可以通过配置hz参数，来加大Redis过期扫描的策略来处理。

默认hz，在10 ~ 100，官方建议，通过压测，建议将其设置成60

