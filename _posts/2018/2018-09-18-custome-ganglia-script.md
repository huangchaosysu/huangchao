---
layout: single
author_profile: true
title: "编写ganglia自定义监控-python"
date: 2018-09-18 14:24:53
# toc: true
tags:
  - linux
  - ganglia
  - python
categories:
  - ganglia
---

### 1.配置modpython

一般在这个路径下/etc/ganglia/conf.d/，创建modpython.conf
```
modules{
    module{
        name = "python_module"
        path = "/usr/lib/ganglia/modpython.so"（这个文件在完整安装完ganglia后会有）
        params="/var/www/ganglia/modpython"(这个参数是ganglia搜寻自定义监控模块的路径)
    }
}

include ('/etc/ganglia/conf.d/*.pyconf')  #pyconf文件是python模块配置文件
```

### 2.用python编写自定义的监控模块

假设我们编写的模块叫temp.py,将这个文件放到上面定义的params目录下(/var/www/ganglia/modpython)  
```
acpi_file = "/proc/acpi/thermal_zone/THRM/temperature"

def temp_handler(name):
    return 10

def metric_init(params):
    global descriptors , acpi_file

    if 'acpi_file' in params:
        acpi_file = params['acpi_file']
    d1 = { 'name':  'temp' ,
        'call_back': temp_handler ,
        'time_max': 90 ,
        'value_type': 'uint' ,
        'units': 'C' ,
        'slope': 'both' ,
        'format': '%u' ,
        'description': 'Temperature of host' ,
        'groups': 'health' }

    descriptors = [d1]
    return descriptors

def metric_cleanup():
    pass

#This code is for debugging and unit testing
if __name__  ==  '__main__':
    metric_init({})
    for d in descriptors:
        v = d['call_back'](d['name'])
        print 'value for %s is %u' % (d['name'], v)
```

监控模块必须有的3个函数  
1. def metric_init(params): 在你的模块中，这个函数必须存在，而且命名一致。他会被在初始化时调用一次，也就是在gmond启动的时候。他可以被用来做收集度量的各种初始化。metric_init() t同时也有一个单字典类型的参数，他包含了在gmond.conf中为这个模块设计的配置指令。除了完成其他的初始化工作之外，metric_init()必须创建，填充，返回这个度量描述字典或者字典列表。每个描述字典都必须包含以下几个元素：  
name:度量名称  
call_back: 在收集度量数据时被调用的函数

如果你的度量模块支持多种度量，每一个都通过他们的自己的度量描述被定义，那么你的模块中就需要实现不止一个的metric_handler 函数。  
time_max:以秒为单位的收集时调用函数的最大时间间隔  
该元素的确切性质还不清楚，因为它关系到你的模块的pyconf配置文件中的“collect_every”配置指令。对于所有意图和目的，这个因素似乎没用。  
value_type: string | uint | float | double  
units: 你的度量单位  
slope: zero | positive | negative | both  
这个值映射到为RRDTool定义的数据源类型
If ‘positive’, RRD file generated will be of COUNTER type (calculating the rate of change)  
如果是’postive’,生成的RRD文件是COUNTER类型的（计算变化的速率）
If ‘negative’, ????（官方的也是如此）
‘both’的话就是GAUGE类型（没有计算，仅仅以报告的值绘图）
metric如果是’zero’，这个度量会被呈现在”Time and String Metrics“ 或者 “Constant Metrics”，取决于度量的单位。  
format: 度量的格式字符串
必须符合value_type，否则你的度量就会未知（参考:http://docs.python.org/library/stdtypes.html#string-formatting）  
description: 度量的描述  
在前台网页中，在划过主机度量图形时被呈现。
groups (optional): 度量的隶属分组  
相同分组的度量在前台网页中会被关联在一起

除了回调函数以外，这些元数基本上和那些需要提供给gmetric命令行工具的数据是同一类型的。可以查看gmetric帮助文档获取更多信息。度量描述符也可以包含额外的属性和值，他们会作为额外的数据附加到度量元数据中。附加数据会被Ganglia本身忽略，不过他可以作为显示或者度量处理数据被用在前台网页中

2. def metric_cleanup():在你的模块中，这个函数必须存在而且命名为’metric_cleanup’。他会在gmond关闭时被调用一次。任意模块的清理代码都可以放在这里，函数不应该有返回值。

3. def metric_handler(name):这个’metric_handler‘函数可以被定义为任何你喜欢的名字，因为他和你定义在度量描述块中的’call_balck’函数想匹配。他有一个参数’name’，就是在你的度量描述块中的name元素.本例中该函数的名称为temp_handler
参数name对应"temp"


模块配置文件pyconf
和这个模块想匹配的配置文件，temp.pyconf，放置于 /etc/ganglia/conf.d/temp.pyconf。看起来像这样：  
```
modules  { 
     module  { 
          name  =  "temp" #这个name的值应该有py文件的名字一致。这里是temp.py，所以这里是temp
          language  =  "python" 
          # The following params are examples only 
          # They are not actually used by the temp module 
          param RandomMax  { 
          value  =  600 
          } 
          param ConstantValue  { 
               value  =  112 
          } 
     } 
} 

collection_group  { 
     collect_every  =  10 
     time_threshold  =  50 
     metric  { 
          name  =  "temp" 
          title  =  "Temperature" 
          value_threshold  =  70 
     } 
}
```

name  
这个名字和你创建的模块名字想对应(.py结尾）  
language  
除非你用C/C++编写你的模块，否则你必须明确的声明模块所用的语言。声明‘python’作为你的语言，告诉gmond到python_modules这个目录里搜索你的模块文件。  
param  
每个param子节点有一个name和一个value，他们组成了name/value对作为参数被传递到上面描述的metric_init()函数中

Collection_group  
配置文件剩下的部分具有相同的格式，collection_group 或者 metric。查阅gmond.conf的帮助文档是很有收获的，不过我们将简单介绍例子中的 collection_group 指令。

collect_every or collect_once
collect_every 告诉 gmond 从定义在collection_group的度量中收集数据的频率（秒为单位）。在例子中，’temp’度量会间隔十秒被收集。你也可以设定collect_once=yes命令gmond收集静态度量，他们在会在gmond启动时被收集一次。这对那些在运行期间不会改变的东西是很有用的（比如运行的CPU个数）

time_threshold  
将度量数据报告给Ganglia的最大频率（秒为单位）。在例子中，temp模块会至少每50秒报告一次。
这个指令会在被收集度量的值大于metirc定义的‘value_threshold’时被抛弃哦。

metric  
这是你定义特殊度量设置的地方

name:特殊度量的名字，也定义在你模块里描述符字典类型上  
title:可选的友好度量名称，将会在Ganglia前台被显示  
value_threshold: 如果收集到的度量报告的值（在你的描述符中定义的单位）超过定义在这里的值，那么它就会报告给ganglia而忽略collection_group中定义的 ‘time_threshold’参数