# 过滤器返回异常信息（返回类型为void时，如何返回给前端异常信息）

```java
/**
 * 封装异常返回数据
 * @param response
 * @param json
 * @throws Exception
 */
private void returnJson(ServletResponse response, String json) throws Exception{
    PrintWriter writer = null;
    response.setCharacterEncoding("UTF-8");
    response.setContentType("text/html; charset=utf-8");
    try {
        writer = response.getWriter();
        writer.print(json);

    } catch (IOException e) {
        log.error("response error",e);
    } finally {
        if (writer != null) {
            writer.close();
        }
    }
}
```