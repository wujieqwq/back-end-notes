# Shell

Shell是Linux操作系统中的一个命令解释器，是一个为用户提供操作页面的程序，它提供了用户与内核交互操作的接口，它有自己的编程语言，即它也是一种程序设计语言。

## 命令类型

Shell命令可以识别用户输入一条命令或多条命令的组合从而完成一项/多项功能操作，它有以下几种类型来执行命令。

1. 单条命令

如下通过ls命令使用长格式显示文件内容，包括文件的属性、权限等数据：

```shell
[yyx@yyx12 ~]$ ls -l
```

2. 串行命令

    串行命令是在命令行中由“;”分号隔开多条命令，Shell执行串行命令时是从左至右执行的，其中各命令互不干扰，它们之间没有逻辑关系，这种方法可以一次执行多个命令操作，执行完后每条命令的输出结果依次显示。

如下分别依次执行ls列出当前目录和文件、pwd显示当前工作目录，然后再通过cd命令将当前工作目录切换至/etc：

```shell
[yyx@yyx12 ~]$ ls;pwd;cd /etc
```

3. 命令组

    串行命令中各命令之间是没有逻辑关系的，而命令组（复合命令），它可以使多条具有一定逻辑关系的命令结合在一起执行，从而形成一个相对独立的整体。

- 使用圆括号的命令组
通过圆括号“()”将多条命令包括起来，从而可将括号内的多条命令可看作一个整体。
例如，我们首先在/home目录下创建一个空文件newfile，通过cat命令显示“Centos系统版本信息”文件内容以及date命令显示当前时间，将这两个命令的结果作为一个整体通过”>“标准输出重定向符，将其赋给/home/newfile，此时newfile文件内容就是这两个命令所显示的内容：

```shell
[yyx@yyx12 ~]$ su root
...
[root@yyx12 yyx]# touch /home/newfile
[root@yyx12 yyx]# (cat /etc/centos-release;date)>/home/newfile
[root@yyx12 yyx]# cat /home/newfile
```

- 使用管道的命令组
管道命令，“|”像一个管子一样连接多条命令，将前一条命令的标准输出重定向到后一条命令作为其标准输入，例如两条命令通过”|“连接，第一条命令的执行结果作为执行的参数传到第二条命令，显示的是第二条命令以第一条命令为参数的结果。
例如，将cat命令和grep命令通过管道“|”命令连接，使用cat命令显示/etc/filesystems的内容，将其作为grep命令的输入，grep命令后跟要查找的名称，使grep查找符合的内容：

```shell
[yyx329@192 ~]$ cat /etc/filesystems|grep xfs
[yyx329@192 ~]$ cat /etc/filesystems|grep ext
```

4. 前台命令/后台命令

- 前台命令与后台命令
放在后台执行的程序（命令）称为后台命令，可以在命令的后面加上“&”符号从而让Shell识别这是一个后台命令，后台命令不用等待该命令执行完成，就可立即接收新的命令，另外后台进程执行完后会返回一个进程号（PID）；相对应的，若Shell执行一条命令并等待该命令结束后才接受新的命令，且该命令运行时默认的输入/输出都在当前终端上，则这个命令称为前台命令。
例如，通过ls 命令列出当前目录信息，将这些信息保存到/home/newfile文件中，由于后面加了“&”，所以不必等待ls命令执行完，也就是这是一个后台命令，最后再通过cat命令打开该文件：

```shell
[yyx@yyx12 ~]$ su root
...
[root@yyx12 yyx]# ls -l > /home/newfile &
...
[root@yyx12 yyx]# cat /home/newfile
```
