# 继承HashMap导致的Json序列化问题

参考文章：https://blog.csdn.net/zry19950714/article/details/115604875

实体类如果继承了HashMap，则在通过fastjson或者通过jackson进行序列化和反序列化的时候，会导致强制的实体类向上转型，从而导致子类属性丢失问题。