**物理知识：**
趋肤效应是指交流电(AC)在导体内部分布的趋势，电流密度在导体表面附近最大，并随着导体深度的增加电流密度的降低而减小。透入深度δ 定义为电流密度仅为表面值的1/e(约37%)的深度，它取决于电流的频率和导体的磁性。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122413413072.png)
ρ=是导体的电阻率，单位为ω/m
f=频率，单位为赫兹
μ=导体的绝对磁导率
绝对磁导率(μ)=μo xμr
μo  = 4π x 10-7 H/m
这里取三种材料，分别为铜，铝和铁
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201224162327354.png)

这里主要以频率为变量，利用python画出变化折线图

**python基本语法：**
根号计算：
注意根号计算不能进行数组的整体计算
```python
import math
math.sqrt( x )
```
指数表示方法：
用**表示指数
```python
p = 1.678*10**-8
```
列表建立：

```python
import numpy as np
f = np.arange(1, 17, 1) * 10**11
```
注意不能这样写
f = np.arange(100, 1700, 1) * 10**9
会出现如下错误：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201224151405177.png)
暂时不知道什么原因

画图：

```
    import matplotlib.pyplot as plt
    l1 = plt.plot(f, y, 'r--', label='copper') #线的名称
    plt.plot(f, y, 'ro-')  # 坐标点
    plt.title('skin depth') # 标题
    plt.xlabel('f/Hz')  # 横坐标 
    plt.ylabel('depth/m') # 纵坐标
    plt.legend() 
    plt.show()
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122416230469.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
