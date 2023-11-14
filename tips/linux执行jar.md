# Linux执行jar

要运行java的项目需要先将项目打包成war包或者jar包，打包成war包需要将war包部署到tomcat服务器上才能运行。而打包成jar包可以直接使用java命令执行。在linux系统中运行jar包主要有以下四种方式。

一、java -jar XXX.jar

这是最基本的jar包执行方式，但是当我们用ctrl+c中断或者关闭窗口时，程序也会中断执行。

二、java -jar XXX.jar &

&代表在后台运行，使用ctrl+c不会中断程序的运行，但是关闭窗口会中断程序的运行。

三、nohup java -jar XXX.jar &

使用这种方式运行的程序日志会输出到当前目录下的nohup.out文件，使用ctrl+c中断或者关闭窗口都不会中断程序的执行。

四、nohup java -jar XXX.jar >temp.out &

\>temp.out的意思是将日志输出重定向到temp.out文件，使用ctrl+c中断或者关闭窗口都不会中断程序的执行。