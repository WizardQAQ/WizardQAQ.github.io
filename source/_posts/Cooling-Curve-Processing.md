---
title: 冷却曲线处理
date: 2019-06-02 18:41:37
tags: Python
---
# 不冷曲线

步冷曲线的获得为使用K型热电偶测量，实验条件为：
>将试样加热至1000℃，采用5、10、20、50和自由冷却的方式冷却至室温
数据处理部分均以20℃/s的为例（其实是感觉只有这个的数据处理能看:(

## 数据处理

对曲线使用了最小二乘法，一阶差分，二阶差分(好像应该写导数)
<!--more-->
### 最小二乘法

对于整体曲线使用最小二乘法进行回归分析（好像没什么意义，找不出想要的线性部分）。代码见下，拟合结果见图

```Python
import numpy as np
from scipy.optimize import leastsq
import matplotlib.pyplot as plt

def fun1(x,p):
    k,b = p
    return k * x + b

def residuals(p, y, x): 
    return y - fun1(x, p)

data = np.loadtxt('CCT20.txt')
y = data[:,1]   #Cgauge
x = data[:,6]   #Temp
p0 = [-1,-1]    #给定初始值
plsq = leastsq(residuals, p0, args=(data[:, 1], data[:, 6]))
print(plsq[0], plsq, sep='\n')  #最终最小二乘法求得的拟合曲线参数

plt.plot(data[:,6], data[:,1])
plt.plot(data[:,6], fun1(data[:,6], plsq[0]))   #拟合曲线和原始数据
```
![LSQ](Figure_1.png)

### 一二阶导

对离散数据点求“导”，希望找到导数不变的部分即原始数据中的线性部分，以此来求出整个冷却过程的中相变点。

```Python
import numpy as np
import matplotlib.pyplot as plt

data = np.loadtxt('CCT20.txt')
y = data[:,1]   #Cgauge
x = data[:,6]   #Temp

dy = np.zeros([4000,2], dtype = float)  #给定初始数组用于储存差分y的数据（如此不负责任的给了4000:)

i = 0   #迭代变量
interval = 30   #这个间隔主要是跳过一些数据点，较少一些“噪声”。
                #对于5℃/s的冷速，获得数据点较多，可能由于自身传感器的噪声，原始数据还好，但求一阶导后整个数据波动十分巨大，
                # 甚至远超过自身所包含的信息，根据Google的方法，可以跳过一些数据点（这个多简单），或者滤波（我感觉，但是好麻烦）
while i < 905 - interval:   #905为数据总数量
    dy[i,0], dy[i,1] = data[i,6], (data[i+interval,1] - data[i,1])/(data[i+interval,6] - data[i,6]) #一阶导数
    i = i + 1

dy2 = np.zeros([4000,2], dtype = float) #同理一阶，这个0数组为了储存一阶导数的导数
k = 0   #迭代变量
interval2 = 10  #同理
while k < 904 - interval:
    dy2[k,0], dy2[k,1] = dy[k,0], ((dy[k+interval2,1] - dy[k,1]) / (dy[k+interval2,0] - dy[k,0]))   #二阶导数
    k += 1

fig, ax1 = plt.subplots()
color = 'tab:red'
ax1.set_xlabel('Temp')
ax1.set_ylabel('CGauge', color=color)
ax1.plot(data[:,6], data[:,1], color=color, label='Temp-CGauge')
ax1.tick_params(axis='y', labelcolor=color)
ax2 = ax1.twinx() #副y轴，考虑到Temp和导数的数据范围差别还是蛮大的所以又用了一根
color = 'tab:blue'
ax2.set_ylabel('D', color=color)  # we already handled the x-label with ax1（抄别人代码的注释
ax2.plot(dy2[:,0], dy2[:,1], color=color, label='2nd D')
color = 'tab:green'
ax1.plot(dy[:,0], dy[:,1], color=color, label='1st D')
ax2.plot([0,1100], [0,0], label='Ref line')
ax2.tick_params(axis='y', labelcolor=color)
ax2.set_ylim([-0.00008,0.00008])
ax1.legend(loc=1)
ax2.legend(loc=2)   #图例的位置，这样两边放好难看，，，，
plt.show()
```

![ProcessingFig](Figure_2.png)

通过图片可以明显看出在相变前，曲线的二阶导在0附近波动（感觉这地方还是处理下会好看点，但这样是不是就算某种程度上修改数据:(。

### 精进？

又改了下代码，把求导数的部分写成了一个函数，方便对不同组数据的使用。代码见下(另不改y轴范围，不对曲线进行处理，这数据能看？？？做啥研究，上哪门子学回家卖红薯把(╬▔皿▔)凸

```Python
import numpy as np
import matplotlib.pyplot as plt

original_data = ['CCT5.txt', 'CCT10.txt', 'CCT20.txt', 'CCT50.txt', 'CCTfree.txt']
CCT5 = np.loadtxt(original_data[0])
CCT10 = np.loadtxt(original_data[1])
CCT20 = np.loadtxt(original_data[2])
CCT50 = np.loadtxt(original_data[3])
CCTfree = np.loadtxt(original_data[4])

def derivative(x, y, interval):
    length_y = len(y)
    dy = np.zeros([length_y, 2], dtype = float)
    i = 0
    i_interval = interval
    while i < length_y - i_interval:   
        dy[i,0], dy[i,1] = x[i], (y[i+i_interval] - y[i])/(x[i+i_interval] - x[i])
        i = i + 1
    return dy
dy_20 = derivative(CCT20[:,6], CCT20[:,1], 20)
ddy_20 = derivative(dy_20[:,0], dy_20[:,1], 20)


plt.plot(ddy_20[:,0], ddy_20[:,1])
plt.show()
```

![Fig 3](Figure_3.png)

