最近，在重构老项目（PHP写的）的时候，之前项目的返回结果集是GBK编码，但是，最近做服务化时，需要同时兼容UTF-8和GBK2种返回格式，故有以下内容解决~

一、Dubbox支持多种编码格式

dubbox使用的rest服务，是当当基于resteasy集成的，故只需要扩展这一部分，即可，所以，只需要我们扩展自定义的Provider（将返回的数据按自定的需求输出）。

1. 在资源目录文件下，创建自定义Provider

```
新建resources/META-INF/services/javax.ws.rs.ext.Providers目录，并添加自定义的Provider的package name
```



