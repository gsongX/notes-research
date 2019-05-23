---
title: 2017-2-10 python 性能分析 cProfile
tags: python, cProfile,
grammar_cjkRuby: true
---

####  Python标准库中提供了三种用来分析程序性能的模块，分别是cProfile, profile和hotshot，另外还有一个辅助模块stats。这些模块提供了对Python程序的确定性分析功能，同时也提供了相应的报表生成工具，方便用户快速地检查和分析结果。 

### 这三个性能分析模块的介绍如下： 

> cProfile：基于lsprof的用C语言实现的扩展应用，运行开销比较合理，适合分析运行时间较长的程序，推荐使用这个模块； 

> profile：纯Python实现的性能分析模块，接口和cProfile一致。但在分析程序时增加了很大的运行开销。不过，如果你想扩展profiler的功能，可以通过继承这个模块实现； 

> hotshot：一个试验性的C模块，减少了性能分析时的运行开销，但是需要更长的数据后处理的次数。目前这个模块不再被维护，有可能在新版本中被弃用。 




## 内置cProfile


```python
def sum_num(max_num):
    total = 0
    for i in range(max_num):
        total += i
    return total
 
 
def test():
    total = 0
    for i in range(40000):
        total += i
 
    t1 = sum_num(100000)
    t2 = sum_num(400000)
 
    return total
 
if __name__ == "__main__":
    import cProfile
 
    # 直接把分析结果打印到控制台
    cProfile.run("test()")
    # 把分析结果保存到文件中
    cProfile.run("test()", filename="result.out")
    # 增加排序方式
    cProfile.run("test()", filename="result.out", sort="cumulative")
```

             6 function calls in 0.028 seconds
    
       Ordered by: standard name
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            2    0.024    0.012    0.024    0.012 <ipython-input-2-691327147ef5>:1(sum_num)
            1    0.003    0.003    0.028    0.028 <ipython-input-2-691327147ef5>:8(test)
            1    0.000    0.000    0.028    0.028 <string>:1(<module>)
            1    0.000    0.000    0.028    0.028 {built-in method builtins.exec}
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
    
    


# 分析工具
## 使用cProfile分析的结果可以输出到指定的文件中，但是文件内容是以二进制的方式保存的，用文本编辑器打开时乱码。所以，Python提供了一个pstats模块，用来分析cProfile输出的文件内容。它支持多种形式的报表输出，是文本界面下一个较为实用的工具。使用非常简单：


```python

import pstats
 
# 创建Stats对象
p = pstats.Stats("result.out")
 
# strip_dirs(): 去掉无关的路径信息
# sort_stats(): 排序，支持的方式和上述的一致
# print_stats(): 打印分析结果，可以指定打印前几行
 
# 和直接运行cProfile.run("test()")的结果是一样的
p.strip_dirs().sort_stats(-1).print_stats()
 
# 按照函数名排序，只打印前3行函数的信息, 参数还可为小数,表示前百分之几的函数信息 
p.strip_dirs().sort_stats("name").print_stats(3)
 
# 按照运行时间和函数名进行排序
p.strip_dirs().sort_stats("cumulative", "name").print_stats(0.5)
 
# 如果想知道有哪些函数调用了sum_num
p.print_callers(0.5, "sum_num")
 
# 查看test()函数中调用了哪些函数
p.print_callees("test")
```

    Fri Feb 10 17:27:19 2017    result.out
    
             6 function calls in 0.021 seconds
    
       Ordered by: standard name
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            2    0.020    0.010    0.020    0.010 <ipython-input-2-691327147ef5>:1(sum_num)
            1    0.001    0.001    0.021    0.021 <ipython-input-2-691327147ef5>:8(test)
            1    0.000    0.000    0.021    0.021 <string>:1(<module>)
            1    0.000    0.000    0.021    0.021 {built-in method builtins.exec}
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
    
    
    Fri Feb 10 17:27:19 2017    result.out
    
             6 function calls in 0.021 seconds
    
       Ordered by: function name
       List reduced from 5 to 3 due to restriction <3>
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    0.000    0.000    0.021    0.021 {built-in method builtins.exec}
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
            1    0.000    0.000    0.021    0.021 <string>:1(<module>)
    
    
    Fri Feb 10 17:27:19 2017    result.out
    
             6 function calls in 0.021 seconds
    
       Ordered by: cumulative time, function name
       List reduced from 5 to 3 due to restriction <0.5>
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    0.000    0.000    0.021    0.021 {built-in method builtins.exec}
            1    0.000    0.000    0.021    0.021 <string>:1(<module>)
            1    0.001    0.001    0.021    0.021 <ipython-input-2-691327147ef5>:8(test)
    
    
       Ordered by: cumulative time, function name
       List reduced from 5 to 1 due to restriction <'test'>
    
    Function                                called...
                                                ncalls  tottime  cumtime
    <ipython-input-2-691327147ef5>:8(test)  ->       2    0.020    0.020  <ipython-input-2-691327147ef5>:1(sum_num)
    
    





    <pstats.Stats at 0x7f204c476518>

