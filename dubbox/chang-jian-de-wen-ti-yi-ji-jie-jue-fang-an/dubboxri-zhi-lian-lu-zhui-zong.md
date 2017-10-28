该篇要说日志链路追踪不是值得各种服务之前调用的一个完整的链路，比如：zipkin等，当然，Spring Cloud都集成了，这篇只是想描述下对于服务端，如何快速定位服务中的问题（在重构中，尤其跨部门之前的调用，经常遇到一些问题，然后，就是各种“甩锅”），那么，这篇将处理，如何在服务端快速定位问题。当然，如果集成那种完整的链路调用，也是可以的，我这篇的初衷是怎么轻量的处理这种问题~

一、Dubbox日志链路追踪

下面，贴出主要实现代码，其实也比较简单，就不多解释了。

1. 过滤处理

```
@Component("traceFilter")
public class TraceFilter implements Filter {

    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        String methodName = invocation.getMethodName();
        if (methodName.equals("ping")) {
            return invoker.invoke(invocation);
        }

        String traceId = generateRandomSed(generateDefaultTraceLogIdPrefix());
        TraceLogUtil.put(traceId);

        return invoker.invoke(invocation);
    }
}
```

  2. 工具类

```
public final class TraceLogUtil {

    private static final ThreadLocal<Map<String, String>> inheritableThreadLocal = new InheritableThreadLocal<>();
    private static final Random random = new Random(System.currentTimeMillis());

    private static final String TRACE_ID = "X-B3-TraceId";

    public static void put(String val) {
        put(TRACE_ID, val);
    }

    public static void put(String key, String val) {
        if (key == null) {
            throw new IllegalArgumentException("key cannot be null");
        }
        Map<String, String> map = inheritableThreadLocal.get();
        if (map == null) {
            map = new HashMap<>();
            inheritableThreadLocal.set(map);
        }
        map.put(key, val);
    }

    public static String get() {
        return get(TRACE_ID);
    }

    public static String get(String key) {
        Map<String, String> map = inheritableThreadLocal.get();
        if ((map != null) && (key != null)) {
            return map.get(key);
        }
        return null;
    }

    public static void remove() {
        remove(TRACE_ID);
    }

    public static void remove(String key) {
        Map<String, String> map = inheritableThreadLocal.get();
        if (map != null) {
            map.remove(key);
        }
    }

    public static void clearTrace() {
        Map<String, String> map = inheritableThreadLocal.get();
        if (map != null) {
            map.clear();
            inheritableThreadLocal.remove();
        }
    }

    public static Random getRandomSed() {
        return random;
    }

    public static String generateDefaultRandomSed() {
        return IdConversion.convertToString(random.nextLong());
    }

    public static String generateRandomSed(String str) {
        return str + "_" + IdConversion.convertToString(random.nextLong());
    }

    public static String generateDefaultTraceLogIdPrefix() {
        return DateUtil.getNowDate2String("yyyyMMdd_HHmmssSSS");
    }

    public static String generateTraceLogIdPrefix(String appId) {
        return appId + "_" + DateUtil.getNowDate2String("yyyyMMdd_HHmmssSSS");
    }

    private static class IdConversion {
        public static String convertToString(long id) {
            return Long.toHexString(id);
        }

        public static long convertToLong(String lowerHex) {
            int length = lowerHex.length();
            if (length >= 1 && length <= 32) {
                int beginIndex = length > 16 ? length - 16 : 0;
                return convertToLong(lowerHex, beginIndex);
            } else {
                throw isntLowerHexLong(lowerHex);
            }
        }

        public static long convertToLong(String lowerHex, int index) {
            long result = 0L;

            for (int endIndex = Math.min(index + 16, lowerHex.length()); index < endIndex; ++index) {
                char c = lowerHex.charAt(index);
                result <<= 4;
                if (c >= 48 && c <= 57) {
                    result |= (long) (c - 48);
                } else {
                    if (c < 97 || c > 102) {
                        throw isntLowerHexLong(lowerHex);
                    }

                    result |= (long) (c - 97 + 10);
                }
            }

            return result;
        }

        static NumberFormatException isntLowerHexLong(String lowerHex) {
            throw new NumberFormatException(lowerHex + " should be a 1 to 32 character lower-hex string with no prefix");
        }
    }
}
```



