项目中，经常出现本身定义好了的wiki接口，但是，调用方可能不按规矩来，这样可能出现意想不到的结果（之前，也遇到过，调用方说调用了，但是服务方确实没有能处理这种情况），所以新增这种过滤，方便服务端查找问题，而且对这种不合法的参数，给以特殊的状态码~

一、Dubbox过滤不合规则的参数

熟悉Dubbo流程的都应该知道，哦们只需要做相应的Interceptor，将不合法的参数，以一状态码返回。

代码中，可能掺杂一些项目上的代码，这个可以依据自己的情况来处理

```
public class JacksonWriterInterceptor implements WriterInterceptor {

    private static final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public void aroundWriteTo(WriterInterceptorContext context) throws IOException, WebApplicationException {
        Object entry = context.getEntity();
        if (entry instanceof BaseDataResult) {
            context.proceed();
            return;
        }

        BaseDataResult result = new BaseDataResult(true);
        result.setErrno(ErrorCodeEnum.ERROR_INVALID_PARAMS.getCode());
        result.setErrmsg(StringUtil.getValue(context.getEntity()));

        this.writeTo(context.getOutputStream(), result);
    }

    private void writeTo(OutputStream outputStream, Object obj) throws IOException {
        JsonGenerator jsonGenerator = objectMapper.getFactory().createGenerator(outputStream);
        jsonGenerator.disable(JsonGenerator.Feature.AUTO_CLOSE_TARGET);

        try {
            objectMapper.writeValue(outputStream, obj);
        } finally {
            if (jsonGenerator != null) {
                jsonGenerator.flush();
            }

            if (!jsonGenerator.isClosed()) {
                jsonGenerator.close();
            }
        }

    }
}

```



