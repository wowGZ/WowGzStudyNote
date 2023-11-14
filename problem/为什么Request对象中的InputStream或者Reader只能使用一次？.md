# 为什么Request对象中的InputStream或者Reader只能使用一次？

Request对象中的输入流和reader都是一次性的，因为其中的底层实现使用到了指针和同步处理，所以是不可回溯的。

所以当我们在切面或者过滤中，如果需要处理请求的数据，那么就需要将数据以一个合理的方式进行拷贝。

**1. 先定义一个过滤器，过滤器自己先配置好哈，然后我们在过滤器的doFilter方法中做以下操作：**

```java
@Override
public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
    try {
        MyHttpServletRequestWrapper requestWrapper = new MyHttpServletRequestWrapper((HttpServletRequest) servletRequest);
        System.out.println("RequestBody:" + requestWrapper.getBodyString());
        filterChain.doFilter(requestWrapper, servletResponse);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

**2. 这是MyHttpServletRequestWrapper类，主要是储存body string，和给request对象塞byte[]：**

```java
public class MyHttpServletRequestWrapper extends HttpServletRequestWrapper {

    private final byte[] body;

    private String bodyString;

    public MyHttpServletRequestWrapper(HttpServletRequest request) throws IOException {
        super(request);
        this.bodyString = StreamUtils.copyToString(request.getInputStream(), Charset.forName("UTF-8"));
        body = bodyString.getBytes("UTF-8");
    }

    public String getBodyString() {
        return this.bodyString;
    }

    @Override
    public BufferedReader getReader() throws IOException {
        return new BufferedReader(new InputStreamReader(getInputStream()));
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {

        final ByteArrayInputStream bais = new ByteArrayInputStream(body);

        return new ServletInputStream() {

            @Override
            public boolean isFinished() {
                return false;
            }

            @Override
            public boolean isReady() {
                return false;
            }

            @Override
            public void setReadListener(ReadListener readListener) {

            }

            @Override
            public int read() throws IOException {
                return bais.read();
            }
        };
    }

}
```

