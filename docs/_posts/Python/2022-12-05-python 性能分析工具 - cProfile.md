---
date: 2022-12-05 14:23
status: public
title: 'python 性能分析工具 - cProfile'
---

# 上手示例

使用比较简单，直接看例子：

```python
import cProfile
import timeline_window

cProfile.run('timeline_window.main()')
```

timeline_window 是我们要测试的模块，timeline_window.main() 是它的主函数。
执行完之后打印如下信息

```shell
C:\Python37\python.exe C:\Users\edison\PycharmProjects\DataViz\performace_test.py 
         865497 function calls (858749 primitive calls) in 2.977 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        4    0.000    0.000    0.000    0.000 <__array_function__ internals>:2(amax)
       19    0.000    0.000    0.001    0.000 <__array_function__ internals>:2(any)
       17    0.000    0.000    0.001    0.000 <__array_function__ internals>:2(argwhere)
        1    0.000    0.000    0.000    0.000 <__array_function__ internals>:2(array_equal)
     1603    0.001    0.000    0.005    0.000 <__array_function__ internals>:2(atleast_1d)
        6    0.000    0.000    0.000    0.000 <__array_function__ internals>:2(atleast_2d)
        6    0.000    0.000    0.000    0.000 <__array_function__ internals>:2(concatenate)
      811    0.001    0.000    0.001    0.000 <__array_function__ internals>:2(copyto)
        7    0.000    0.000    0.000    0.000 <__array_function__ internals>:2(empty_like)
      ...
```

每一行依次列出了各个子函数的运行分析信息：

- ncalls 调用次数

- tottime 在给定函数中花费的总时间（不包括调用子函数的时间）

- percall tottime除以ncalls的商

- cumtime 是在这个函数和所有子函数中花费的累积时间（从调用到退出）。

- percall是cumtime除以原始调用次数的商

- filename:lineno(function) 提供每个函数的各自信息
  
  # 保存性能数据

```python
import cProfile
import timeline_window

cProfile.run('timeline_window.main()', 'restats')
```

性能数据的文件会保存到当前目录下的restats文件中。

# 查看性能数据

加载这些数据，可以进行后续的比较分析。

```python
import pstats
from pstats import SortKey

# 加载保存到restats文件中的性能数据
p = pstats.Stats('restats')

# 打印所有统计信息
p.strip_dirs().sort_stats(-1).print_stats()
```

# 查看耗时最多的子函数

最常用的是，查看耗时最多的函数排序，比如前十个：

```python
# 打印累计耗时最多的10个函数
p.sort_stats(SortKey.CUMULATIVE).print_stats(10)

# 打印内部耗时最多的10个函数（不包含子函数）
p.sort_stats(SortKey.TIME).print_stats(10)
```

```python
Mon Dec  5 16:20:20 2022    restats

         42734012 function calls (42727371 primitive calls) in 43.122 seconds

   Ordered by: cumulative time
   List reduced from 2541 to 10 due to restriction <10>

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
    290/1    0.002    0.000   43.589   43.589 {built-in method builtins.exec}
        1    0.026    0.026   43.589   43.589 timeline_window.py:153(main)
        1    0.000    0.000   40.626   40.626 timeline_window.py:141(load_events_from_file)
        1    0.000    0.000   39.737   39.737 timeline_window.py:131(add_timestamps_scatter)
        1    0.010    0.010   39.722   39.722 timeline_scatter.py:80(set_events)
      801    0.005    0.000   39.419    0.049 PlotItem.py:597(addLine)
      803    0.012    0.000   39.309    0.049 PlotItem.py:521(addItem)
      808    0.008    0.000   39.286    0.049 ViewBox.py:402(addItem)
     1628    0.205    0.000   39.004    0.024 ViewBox.py:896(updateAutoRange)
     3231   10.849    0.003   38.744    0.012 ViewBox.py:1404(childrenBounds)
```

于是，我们找到了耗时的大头：      调用了 801 次耗时 39ms 的 PlotItem.py:597(addLine) 函数。

# 参考链接

https://blog.csdn.net/Bit_Coders/article/details/120154767