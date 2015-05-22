## Homework: shell

这个作业通过去实现一些shell的特性去完成一个小的shell程序，来让我们更加的熟悉Unix系统调用接口,这个小的shell程序，被称之为6.828 shell

你可以在任何支持Unix API的(操作系统上)来完成这件任务(作业)（一台Linux Athena机器，装了Linux或MacOS系统的笔记本电脑等）。最后以文件的形式提交你的6.828shell到[submission web site](https://ccutler.scripts.mit.edu/6.828/handin.py/)，(并)将其命名为"hwN.c"，N代表我们进度中的家庭作业的号码。

阅读[xv6 book](http://pdos.csail.mit.edu/6.828/2014/xv6/book-rev8.pdf)第0章.

如果你对shell的工作原理不熟悉，那么就从[Unix hands-on](http://web.mit.edu/6.033/www/assignments/handson-unix.html)中的6.033开始做。

下载并阅读[6.828 shell](http://pdos.csail.mit.edu)，你会发现6.828shell包含两个部分：解析和使用shell命令，编译器只能识别像下面的一些简单的shell命令：

``ls > y``

``cat < y | sort | uniq | wc > y1``

``cat y1``

``rm y1``

``ls |  sort | uniq | wc``

``rm y``

把这些命令复制到t.sh文件中

要编译sh.c，我们必须要有C语言的编译器，像gcc，在Athena中你可以输入下面命令：

``$ add gnu``

如果你使用的是自己的电脑，为了使用gcc编译器，你不得不安装gcc编译器，一旦你有gcc编译器，那么你可以通过下面的命令来编译你的文件：

``$ gcc sh.c``

编译通过就会生成一个a.out的文件，你可以通过下面的命令来运行a.out这个文件：

``$ ./a.out < t.sh``

执行结果会输出一些错误信息，因为你没有实现一些特性，在这个作业的其余部分，你将实现这些功能。

### 执行一些简单的命令

执行下面的一些简单命令，像：

``$ ls``

解析器已经为用户建立一个exec命令，*so the only code you have to write is for the ' ' case in runcmd.* 你可能会发现查看exec手册页非常有用，你可以输入``man 3 exec``命令来查看关于execv的介绍，当execv失败时打印错误信息。
测试你的程序，运行你的编译结果a.out文件：

``6.828$./a.out``

输入上面命令后将打印出提示并等待用户的输入，sh.c输出6.828$提示，以防止你将6.828shell和系统自带的shell混淆，下面输入你系统自带的shell：

``6.828$ ls``

输入系统自带的shell后会打印出错误（除非在我们执行的目录下有一个命为ls的程序），下面继续输入系统自带的shell：

``6.828$ /bin/ls``

上面的命令会执行``/bin/ls``程序，这个命令会打印出我们现在工作目录下文件的名字。我们可以输入ctrl+D来停止6.828shell，这样我们就能回到了我们系统自带的shell环境下了。
如果我们需要的程序不在当前工作的目录下，我们就不得不对每个程序都输入``/bin``来改变6.828shell，如果你对这很感兴趣，你可以自己实现一个PATH变量支持。
### I/O redirection
我们可以输入下面的命令来实现I/O的重定向：
``echo "6.828 is cool" > x.txt
cat < x.txt``
解析器能识别出``>``和``<``，并且能为用户建立一个重定向的命令，所以我们的工作仅仅是补全那些在使用重定向符号所需要的命令。你可能会发现open和close的man手册条目非常有用。
必须确保当我们使用系统调用失败时能打印出错误信息。
必须确保你按照上面测试输入能够实现正确运行。一个常见的错误就是忘记指定必须创建的文件的权限（即open的第三个参数）。

### Implement pipes
实现管道后我们可以运行类似下面的管道命令：

``$ ls | sort | uniq | wc``

解析已经能识别``|``符号，并且为用户建立管道命令，因此我们只需要在运行管道命令的时候写上|，你会发现使用man手册能找到很多对pipe, fork, close, dup等有用的信息。

测试你能否运行上面的管道命令，排序程序可能会在目录``/usr/bin``下，在这种情况下，你可以键入绝对路径的``/usr/bin/sort``来运行排序程序（你可以在系统自带的shell中输入which sort找出sort可执行文件在shell搜索路径中的哪个目录中）。
现在你可以正确输入下面的命令了：

``6.828$ a.out < t.sh``

必须确保你使用的是这个程序正确的绝对路径。
不要忘记把你们的解答提交到[submission web site](https://ccutler.scripts.mit.edu/6.828/handin.py/)，可以带也可以不带拔高题。
### Challenge exercises
你可以增加你自己的shell的功能，但是，你可以考虑下面的建议作为你下一步的开始：

* 可以执行多个shell命令，各个命令直接用``;``隔开
* 通过执行"("and")"来执行子shell
* 通过借助"&"和"wait"来实现在后台运行的命令

所有的这些需求都需要改变解析器和runcmd函数