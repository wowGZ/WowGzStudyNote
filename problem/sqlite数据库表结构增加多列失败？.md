# sqlite数据库表结构增加多列失败？

今天同事问了个问题，发了个语句说这个有错么，我第一反应是没错，但是他那边报了执行错误。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22461487/1699930617151-3485e0d6-769c-43fe-9dbd-57a33bf35b51.png)

出于好奇，详细的问了一下，原来是sqlite数据库，本着mysql大法好的我先入为主的认为是mysql了，这个语句在mysql中执行自然是没错的，但是sqlite数据库执行却出现了问题。

经过阅读部分博客的帖子以及在sqlite官方网站检索阅读相关文档后，得出结论：**sqlite目前是不支持一条alter table语句插入多列的**

大家可以去[官方文档](https://www.sqlite.org/lang_altertable.html)中查看，如图所示，`alter table`的语句流程图，解析完成一个列的添加之后就已经结束了，所以目前想要新增多列的话，只能自行多次执行`alter table`来实现了。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22461487/1699930776678-dfdc1725-8cb5-4b96-a28a-2b49032ea4d6.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)