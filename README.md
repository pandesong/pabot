# pabot

[Pabot](https://github.com/mkorpela/pabot.git) 在此项目基础上拉取的一个分支进行修改，可以指定测试套进行用例分割

## 安装

克隆此存储库并运行：

     setup.py  install


## 命令行选项

支持所有Robot Framework命令行选项以及以下选项（这些选项必须在普通RF选项之前）：

--verbose     
  来自并行执行的更多输出

--spilt
  指定需要并行分割执行的测试套


--testlevelsplit          
  在测试级别而不是默认套件级别上拆分执行。
  如果.pabotsuitenames包含测试和套件，那么这个
  只会影响新套件并仅拆分它们。
  当套房和测试中都留下这个标志
  .pabotsuitenames文件也只会影响新的套件和
  将它们添加为套件文件。

--command [开始执行Robot Framework的实际命令] --end-command    
  Robot Framework脚本适用于不直接使用pybot的情况

--processes   [进程数]          
  要使用多少个并行执行程序（默认最大值为2和cpu计数）

--pabotlib          
  启动PabotLib远程服务器。 这样可以在并行测试执行之间进行锁定和资源分配。

--pabotlibhost   [主机名]          
  PabotLib远程服务器的主机名（默认为127.0.0.1）
  如果与--pabotlib选项一起使用，将更改创建的远程服务器的主机侦听地址（请参阅https://github.com/robotframework/PythonRemoteServer）
  I如果在没有--pabotlib选项的情况下使用，将连接到给定主机中已运行的PabotLib远程服务器实例。 远程服务器也可以与pabot实例分开启动和执行：
  
      python -m pabot.PabotLib <path_to_resourcefile> <host> <port>
      python -m pabot.PabotLib resource.txt 192.168.1.123 8271
  
  这样就可以与多个Robot Framework实例共享资源。

--pabotlibport   [港口]          
  PabotLib远程服务器的端口号（默认为8270）
  有关更多信息，请参阅--pabotlibhost

--resourcefile   [文件路径]          
  可以包含用于分发资源的共享变量的文件的指示符。 这需要与pabotlib选项一起使用。 资源文件语法与Windows ini文件相同。 其中section是一组共享的变量。

--argumentfile[整数]   [文件路径]          
  使用多个[参数文件](http://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#argument-files)选项运行相同的套件。
  例如：

     --argumentfile1 arg1.txt --argumentfile2 arg2.txt

--suitesfrom   [文件路径]          
  （可选）从output.xml文件中读取套件。 套件将运行失败
   第一个和更长的运行将在较短的之前执行。

示例用法：

     pabot test_directory
     pabot --exclude FOO directory_to_tests
     pabot --command java -jar robotframework.jar --end-command --include SMOKE tests
     pabot --processes 10 tests
     pabot --pabotlibhost 192.168.1.123 --pabotlibport 8271 --processes 10 tests
     pabot --pabotlib --pabotlibhost 192.168.1.111 --pabotlibport 8272 --processes 10 tests

### PabotLib

pabot.PabotLib提供的关键字有助于执行程序进程之间的通信和数据共享。
当您必须确保只有一个进程使用某些数据或一次在被测系统的某些部分上运行时，这些可能会有所帮助。

Docs are located at https://mkorpela.github.io/PabotLib.html

### PabotLib示例：

test.robot

      *** Settings ***
      Library    pabot.PabotLib
      
      *** Test Case ***
      Testing PabotLib
        Acquire Lock   MyLock
        Log   This part is critical section
        Release Lock   MyLock
        ${valuesetname}=    Acquire Value Set  admin-server
        ${host}=   Get Value From Set   host
        ${username}=     Get Value From Set   username
        ${password}=     Get Value From Set   password
        Log   Do something with the values (for example access host with username and password)
        Release Value Set
        Log   After value set release others can obtain the variable values

valueset.dat

      [Server1]
      tags=admin-server
      HOST=123.123.123.123
      USERNAME=user1
      PASSWORD=password1
      
      [Server2]
      tags=server
      HOST=121.121.121.121
      USERNAME=user2
      PASSWORD=password2

      [Server2]
      tags=admin-server
      HOST=222.222.222.222
      USERNAME=user3
      PASSWORD=password4


pabot call

      pabot --pabotlib --resourcefile valueset.dat test.robot

### 控制执行顺序和并行度

pabotsuitenames文件包含将要执行的套件列表。
如果还没有在pabot执行期间创建文件。
该文件是pabot在重新执行相同测试时使用的缓存，以加快处理速度。
可以部分手动编辑此文件。
前4行包含不应编辑的信息 - 当某些内容发生变化时，pabot会编辑这些信息。
在此之后，套房命名。

影响执行有三种可能性：

    * 套房的顺序可以更改。
    * 如果应按顺序执行目录（或目录结构），请将目录套件名称添加到行中。
    * 您可以添加一行文本```#WAIT```以强制执行程序等待所有先前的套件执行完毕。


### 全局变量

Pabot会将以下全局变量插入到Robot Framework命名空间中。 这些是为了启用PabotLib功能和自定义侦听器等来获取有关pabot整体执行的一些信息。

      PABOTLIBURI - 它包含正在运行的PabotLib服务器的URI
      PABOTEXECUTIONPOOLID - 它包含当前Robot Framework执行程序的池ID（整数）。 例如，当从您自己的侦听器可视化执行流时，这很有用。
 
