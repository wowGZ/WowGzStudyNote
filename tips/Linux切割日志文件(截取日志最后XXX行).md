# Linux切割日志文件(截取日志最后XXX行)

```sh
tail -100000 XXX.log > AAA.txt
```

获取日志文件 XXX.log 的最后10万行并且生成一个新的日志文件 AAA.txt