---
title: 使用Python绘制XRD数据图(demo)
date: 2019-04-10 20:59:56
tags: Python
---

计划使用Python绘制XRD图，避免一遍又一遍的绘制改图的参数。但是（总有个但是）发现整体实现下来颇为复杂つ﹏⊂

demo代码见下，绘出的图见下（完全以一个门外汉的思维写代码*。*

```Python
import numpy
import matplotlib.pyplot as plt
from scipy.signal import savgol_filter

a = numpy.loadtxt('1.txt')
b = numpy.loadtxt('2.txt')
c = numpy.loadtxt('3.txt')
d = numpy.loadtxt('4.txt')
theta1 = a[:,0]
theta2 = b[:,0]
theta3 = c[:,0]
theta4 = d[:,0]
i1 = a[:,1] + 600
i2 = b[:,1] + 400
i3 = c[:,1] + 200
i4 = d[:,1]
i1 = savgol_filter(i1, 7, 3) #savgol——filter滤波
i2 = savgol_filter(i2, 7, 3)
i3 = savgol_filter(i3, 7, 3)
i4 = savgol_filter(i4, 7, 3)
#rc('axes', linewidth=2)#改变图框线宽
plt.axis([20, 80, -7, 3000])
plt.xlabel("Theta",fontsize=13,fontweight='bold')
plt.ylabel("I",fontsize=13,fontweight='bold')

specific_spectrum_aluminum = numpy.array([38.472, 44.738, 65.133, 78.227])
peak_data_in_xrd = []
def find_peak_near_data(degree_each, xrd_data=[]): #寻峰
    j = 0
    low_bdry = degree_each - 2
    while xrd_data[j, 0] != low_bdry:
        j = j + 1
    '''
    调试
    else:
        print(j)
        print(data_1[j:j+200, 1])
    '''
    return max(xrd_data[j:j+200, 1]) #此处返回值为xrd数据上下一定范围的数据点为了后面找出峰。
                                     #200为4°（上下2°范围，然后除以步长0.02°得到的

m = 0 #m为xrd数据所包含目标相的特征谱线根数（注意数组的起始为0）
while m < 4:
    lb = int(round(specific_spectrum_aluminum[m]))
    peak_data_in_xrd.append(find_peak_near_data(lb, a) + 600)
    m = m + 1
plt.scatter(specific_spectrum_aluminum, peak_data_in_xrd, marker='*', label='Al', s=100)
plt.legend(loc='upper right', title = 'Phases')


plt.plot(theta1,i1,linewidth=1)
#plt.plot(theta2,i2,linewidth=1)
plt.plot(theta3,i3,linewidth=1)
plt.plot(theta4,i4,linewidth=1)

#plt.plot(theta1, a[:,1])
plt.show()
```

![Output](Figure_1.png)