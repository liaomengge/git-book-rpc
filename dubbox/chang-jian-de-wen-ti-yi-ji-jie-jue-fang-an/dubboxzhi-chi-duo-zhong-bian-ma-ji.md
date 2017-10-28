最近，在重构老项目（PHP写的）的时候，之前项目的返回结果集是GBK编码，但是，最近做服务化时，需要同时兼容UTF-8和GBK2种返回格式，故有以下内容解决~

一、Dubbox支持多种编码格式

dubbox使用的rest服务，是当当基于resteasy集成的，故只需要扩展这一部分，即可，所以，只需要我们扩展自定义的Provider（将返回的数据按自定的需求输出）。

1、在资源目录文件下，创建自定义Provider

```
新建resources/META-INF/services/javax.ws.rs.ext.Providers目录，并添加自定义的Provider的package name

如：cn.sh.pdxq.jerry.extension.JacksonProvider
```

2、编辑自定义的Provider

此处是以Unicode兼容GBK和UTF-8来处理的

```
@Provider
@Produces({"application/json", "application/*+json", "text/json"})
public class JacksonProvider extends JacksonJsonProvider {

    public JacksonProvider() {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
        objectMapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
        objectMapper.setTimeZone(TimeZone.getDefault());

        objectMapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);
        //使Jackson JSON支持Unicode编码非ASCII字符
        SimpleModule module = new SimpleModule();
        module.addSerializer(String.class, new StringUnicodeSerializer());
        objectMapper.registerModule(module);
        //设置null值不参与序列化(字段不被显示)
        objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);

        setMapper(objectMapper);
    }

    class StringUnicodeSerializer extends JsonSerializer<String> {

        private final char[] HEX_CHARS = "0123456789ABCDEF".toCharArray();
        private final int[] ESCAPE_CODES = CharTypes.get7BitOutputEscapes();

        private void writeUnicodeEscape(JsonGenerator gen, char c) throws IOException {
            gen.writeRaw('\\');
            gen.writeRaw('u');
            gen.writeRaw(HEX_CHARS[(c >> 12) & 0xF]);
            gen.writeRaw(HEX_CHARS[(c >> 8) & 0xF]);
            gen.writeRaw(HEX_CHARS[(c >> 4) & 0xF]);
            gen.writeRaw(HEX_CHARS[c & 0xF]);
        }

        private void writeShortEscape(JsonGenerator gen, char c) throws IOException {
            gen.writeRaw('\\');
            gen.writeRaw(c);
        }

        @Override
        public void serialize(String str, JsonGenerator gen,
                              SerializerProvider provider) throws IOException {
            int status = ((JsonWriteContext) gen.getOutputContext()).writeValue();
            switch (status) {
                case JsonWriteContext.STATUS_OK_AFTER_COLON:
                    gen.writeRaw(':');
                    break;
                case JsonWriteContext.STATUS_OK_AFTER_COMMA:
                    gen.writeRaw(',');
                    break;
                case JsonWriteContext.STATUS_EXPECT_NAME:
                    throw new JsonGenerationException("Can Not Write String Value Here", gen);
            }
            gen.writeRaw('"');//写入JSON中字符串的开头引号
            for (char c : str.toCharArray()) {
                if (c >= 0x80) {
                    writeUnicodeEscape(gen, c); // 为所有非ASCII字符生成转义的unicode字符
                } else {
                    // 为ASCII字符中前128个字符使用转义的unicode字符
                    int code = (c < ESCAPE_CODES.length ? ESCAPE_CODES[c] : 0);
                    if (code == 0) {
                        gen.writeRaw(c); // 此处不用转义
                    } else if (code < 0) {
                        writeUnicodeEscape(gen, (char) (-code - 1)); // 通用转义字符
                    } else {
                        writeShortEscape(gen, (char) code); // 短转义字符 (\n \t ...)
                    }
                }
            }
            gen.writeRaw('"');//写入JSON中字符串的结束引号
        }

    }
}
```



