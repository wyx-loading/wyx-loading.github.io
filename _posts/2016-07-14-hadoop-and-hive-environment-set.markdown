---
layout: post
title:  Hadoop + Hive 安装记录
date:   2016-07-14 15:38:00 +0800
categories: ['blog', 'hadoop']
---

## windows下安装hadoop 2.6.4

参照 [Build and Install Hadoop 2.x or newer on Windows][Build and Install Hadoop 2.x or newer on Windows]

---
<br/>
按照wiki执行到

`hdfs name node -format`

这一步时，发现 winutils not found 报错。google一下发现是缺少winutils.exe和hadoop.dll等文件。  
传送门：[steveloughran winutils](https://github.com/steveloughran/winutils)  
里面有多个版本对应的文件，将bin覆盖到%HADOOP_PREFIX%\bin即可

再按照wiki的步骤，即可安装成功。


## windows下安装hive 2.1.0

同样是跟着 [wiki][hive wiki getting started] 操作。

---
<br/>
当我开心地第一次执行  hive  时，我的内心是崩溃的，有一大堆报错。基本上就是跟metastore有关，类似这种东东：  

{% highlight console%}
Required table missing : "VERSION" in Catalog Schema
{% endhighlight %}

逛一下stackoverflow，还是有挺多类似这个问题的，挑选一个觉得回答比较详细的  
[http://stackoverflow.com/questions/35655306/hive-installation-issues-hive-metastore-database-is-not-initialized](http://stackoverflow.com/questions/35655306/hive-installation-issues-hive-metastore-database-is-not-initialized)  


感觉解决方案就是手动执行schematool生成一遍数据，看下schematool是shell脚本，尴尬。无奈，到处找找问题所在（everything查查还有没有schematool？）  
答案是有的！！！就在ext下有一堆的cmd，其中就有schemaTool.cmd。看下代码

{% highlight console %}
set CLASS=org.apache.hive.beeline.HiveSchemaTool
set HIVE_OPTS=
pushd %HIVE_LIB%
for /f %%a IN ('dir /b hive-beeline-*.jar') do (
    set JAR=%HIVE_LIB%\%%a
)
popd

if [%1]==[schematool_help] goto :schematool_help

:schematool
    call %HIVE_BIN_PATH%\ext\util\execHiveCmd.cmd %CLASS%
goto :EOF

:schematool_help
    call %HIVE_BIN_PATH%\ext\util\execHiveCmd.cmd %CLASS% --help
goto :EOF
{% endhighlight %}

看上去不像是直接执行的，HIVE_OPTS为空对吧。到bin下看看hive.cmd的代码吧。搜schematool，嗯？？

![hive.cmd代码截图](/assets/schematool-search.png)

perfect，这段东西看上去很眼熟，跟ext文件夹下的脚本名对应上了。
hive --help看看

![hive --help截图](/assets/hive-help.png)

service？？？$SERVICE_LIST？？？大概猜测就是选择对应的脚本吧。结合stackoverflow的答案，执行

`hive --service schematool -initSchema -dbType derby`

我这里选择derby由于我只是先本机测试一下，尝试使用一下hive，所以derby也足够了，当然也可以选择mysql。

之后一路完美执行，再跑hive也能成功了。按照 [wiki Getting Started][hive wiki getting started] 可以尝试简单的hive使用，也可以看HDDL的wiki自己操作一些“高等级”的操作

[Hive Data Definition Language](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL)


最后，感慨一下，对这类工具还是不够熟悉，简单的安装过程也遇到这么多问题。其实是跟在windows下安装有关（赶紧转投Linux的怀抱吧）。

当然对于Hadoop，更完美的是在*nix系统下安装，因为还有其他相关项目，如Ambari™就不支持*nix外的环境。

过程中发现，有一个好的包装，[Hortonworks Sandbox](http://zh.hortonworks.com/products/sandbox/#install)，几乎整个hadoop生态系统都在这个里面了。只不过下载的镜像文件太大，所以选择自己安装，毕竟只是简单的尝试，感觉hddl看上去跟sql这么像，但是patition这种这么高大上的关键字都还没见过呢。

当时刚听同学说hadoop这个东西，还好奇大概是会有一些把一些常用mapReduce操作封装成sql形式的工具的吧。现在好了，实力证明：

**孤陋寡闻**


[Build and Install Hadoop 2.x or newer on Windows]: http://wiki.apache.org/hadoop/Hadoop2OnWindows
[hive wiki getting started]: https://cwiki.apache.org/confluence/display/Hive/GettingStarted