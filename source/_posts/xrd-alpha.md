---
title: 使用Python绘制XRD数据图(alpha)
date: 2019-04-12 00:00:06
tags: Python
---
装一下，给自己点压力つ﹏⊂
>“你们用了半个世纪的时间，让一截三毫米的头发移动了两厘米”程心回答

>“是空间曲率驱动使它移动的“维德说

重新思考了一下整个程序要干什么，需要输入那些数据，需要对这些数据做什么操作，尝试去把这些操作函数化，并且尝试泛化一些参数（不过目前的范围还是限定在铝合金中一些常见相的标定，感觉目前去写pdf卡片的接口有点不现实）。目前的设想是把每个相打包成一个模块，需要要的话直接导入，这样感觉代码可能会更加可读(抑或将每个相向axes添加曲线的过程函数化)。好像整个project的架构还是没清楚つ﹏⊂

目前的代码如下：

```Python
import numpy
import matplotlib.pyplot as plt 
from scipy.signal import savgol_filter

'''
'''
def find_peak_near_data(degree_each, xrd_data=[], xrd_step = 0.02): #这里目前设想再传入一个参数为扫描角度范围
    j = 0
    low_bdry = round(degree_each) - 1
    while xrd_data[j, 0] != low_bdry:
        j = j + 1
    indice = numpy.where(max(xrd_data[j:j+200, 1]) == xrd_data[j:j+200, 1])
    return xrd_data[indice[0] + j, 0], max(xrd_data[j:j+200, 1])#此处返回值为xrd数据上下一定范围的数据点为了后面找峰

data_1 = numpy.loadtxt('2.txt')

maker_al = numpy.empty(shape = (4,2)) #4为特征谱线的个数，这里能不能把写进函数里，利用角度范围确定；又或者将这一块代码函数化
specific_spectrum_aluminum = numpy.array([38.472, 44.738, 65.133, 78.227])
i = 0 
while i < 4:
    maker_al[i, 0], maker_al[i, 1] = find_peak_near_data(specific_spectrum_aluminum[i], data_1)
    i += 1

maker_si = numpy.empty(shape = (4,2)) #4为特征谱线的个数，这里能不能把写进函数里，利用角度范围确定；又或者将这一块代码函数化
specific_spectrum_silicon = numpy.array([28.442, 47.302, 56.121, 69.130])
j = 0
while j < 4:
    maker_si[j, 0], maker_si[j, 1] = find_peak_near_data(specific_spectrum_silicon[j], data_1)
    j += 1

plt.xlabel("Theta",fontsize=13,fontweight='bold')
plt.ylabel("I",fontsize=13,fontweight='bold')
plt.plot(data_1[:, 0], data_1[:, 1])
plt.scatter(maker_al[:,0], maker_al[:, 1] + 20, marker='*', label='Al', s=100)
plt.scatter(maker_si[:,0], maker_si[:, 1] + 20, marker='o', label='Si', s=100)
plt.legend(loc='upper right', title = 'Phases')#图例感觉不太好看需要修改
plt.show()
```
![Output](Figure_1.png)