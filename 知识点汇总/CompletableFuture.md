# JUC：CompletableFuture使用详解



> 参考：https://www.liaoxuefeng.com/wiki/1252599548343744/1306581182447650



对CompletableFuture的理解，或者说是对Java异步非阻塞的理解：

CompletableFuture就是Java世界当中异步非阻塞的标准是闲了

同步非阻塞的方式就像是我们使用线程池 ，submit了一个Runnable或者是Callable，得到一个Future对象，这时候主线程需要完成：要么调用阻塞方法`get()`，要么定时调用isDone方法来查看是否执行完成，最后都需要主线程来做同步操作，但是使用1.8的CompletableFuture，我们可以在其中指定回调函数，这样主线程在submit任务之后就可以不用管了，剩下的后置处理交给CompletableFuture自己去处理



核心处理逻辑：

`CompletableFuture`可以指定异步处理流程：

- `thenAccept()`处理正常结果；
- `exceptional()`处理异常结果；

> 一定要去使用这个方法，不然

- `thenApplyAsync()`用于串行化另一个`CompletableFuture`；
- `anyOf()`和`allOf()`用于并行化多个`CompletableFuture`。





两个小作业：

```java
/**
 * @author LuckyCurve
 * 完成对两个CSV的读取
 */
public class Application1 {
    public static void main(String[] args) throws InterruptedException {

        List<String> list = Collections.synchronizedList(new LinkedList<>());

        CompletableFuture<Void> future1 = CompletableFuture.runAsync(() -> {
            CloseableHttpClient client = HttpClients.createDefault();

            HttpGet request = new HttpGet("https://gist.githubusercontent.com/CatTail/18695526bd1adcc21219335f23ea5bea/raw/54045ceeae6a508dec86330c072c43be559c233b/movies.csv");

            CloseableHttpResponse response = null;

            try {
                response = client.execute(request);

                String[] array = EntityUtils.toString(response.getEntity()).split("\n");

                list.addAll(Arrays.asList(array));
            } catch (IOException e) {
                e.printStackTrace();
            }
        });

        CompletableFuture<Void> future2 = CompletableFuture.runAsync(() -> {
            CloseableHttpClient client = HttpClients.createDefault();

            HttpGet request = new HttpGet("https://gist.githubusercontent.com/CatTail/18695526bd1adcc21219335f23ea5bea/raw/54045ceeae6a508dec86330c072c43be559c233b/movies.csv");

            CloseableHttpResponse response = null;

            try {
                response = client.execute(request);

                String[] array = EntityUtils.toString(response.getEntity()).split("\n");

                list.addAll(Arrays.asList(array));
            } catch (IOException e) {
                e.printStackTrace();
            }
        });

        CompletableFuture.allOf(future1, future2).thenRun(() -> {
            System.out.println(list);
            System.out.println(list.size());
        });

        TimeUnit.SECONDS.sleep(100);
    }
}

```

```java
/**
 * @author LuckyCurve
 * 请求体当中包含下一个需要请求的链接，需要返回其中这个链接的内容
 */
public class Application2 {

    static final String URL = "https://gist.githubusercontent.com/CatTail/18695526bd1adcc21219335f23ea5bea/raw/54045ceeae6a508dec86330c072c43be559c233b/movies.csv";

    public static void main(String[] args) throws InterruptedException {
        CompletableFuture.supplyAsync(() -> {
            CloseableHttpClient client = HttpClients.createDefault();

            CloseableHttpResponse response = null;
            try {
                response = client.execute(new HttpGet(URL));

                return URL;
            } catch (IOException e) {
                e.printStackTrace();
                return null;
            }
        }).thenApplyAsync(s -> {
            CloseableHttpClient client = HttpClients.createDefault();

            CloseableHttpResponse response = null;
            try {
                response = client.execute(new HttpGet(s));

                return EntityUtils.toString(response.getEntity());
            } catch (IOException e) {
                e.printStackTrace();
                return null;
            }
        }).thenAccept(System.out::println).exceptionally(throwable -> {
            throwable.printStackTrace();
            return null;
        });


        TimeUnit.SECONDS.sleep(100);
    }
}

```

