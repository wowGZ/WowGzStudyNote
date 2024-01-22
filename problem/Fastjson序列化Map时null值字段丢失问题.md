# Fastjson序列化Map时Null值丢失的问题

在开发一个功能的时候，发现了一个问题。先来简单说一下背景，因为要实现的功能没有具体的实体，所以后端这边多以Map的方式处理对象。

所以就在开发过程中出现了如下情况，即前端传来的参数，需要经历序列化和反序列化转为我们需要的数据结构。

然后，下面这个图片就是出现问题的情况，再将saveDataArr转为saveDataList的时候，saveDataArr中某对象的成员变量的值发生了丢失。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/22461487/1705495865458-fba3a0f4-8db6-40f4-a5af-d2719eb210ec.png)

举个例子

>saveDataArr     [{"notNull":"asdfwer","nullvalue":null}]
>
>saveDataList    [{"notNull":"asdfwer"}]

经过之前的代码处理之后，对象中值为null的变量都被省略了。这个问题在有时候是不能接受的，当然我现在需要的逻辑就是不能接受的，所以需要处理一下。

经过对fastjson中的代码进行阅读，发现了这个问题的原因，fastjson在序列化map对象时候的**默认策略**会忽略值为null的字段，如下图所示，当值为null的时候如果没有指定对应的策略就不进行处理，也就会出现之前null值字段丢失的问题。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/22461487/1705496427039-889ec605-c6c2-49a2-b5c1-3f23ca2e0525.png)

那么想处理问题就很简单了，我们不用默认策略就可以了，可以指明序列化的策略，如下图这样就不会丢失值为null的字段了。（注：这里使用的fastjson1.X，如果使用2.X的话策略类有所修改，可以自行了解一下）

![image.png](https://cdn.nlark.com/yuque/0/2024/png/22461487/1705496366863-f3881192-0a2a-4ff9-aabb-b8177c6f8b68.png)

