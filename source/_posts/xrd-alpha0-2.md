---
title: 使用Python绘制XRD数据图(alpha)
date: 2019-04-16 18:13:02
tags: Python
---

目前还差一个pdf文件接口的函数，但是对于一些金属间化合物相的标定仍然不太清楚，个别相有许多特征谱线，但是有些峰在xrd实验数据上并不明显，需要人为，或者加一个衍射强度I的判断。

```Python
import numpy
import matplotlib.pyplot as plt 
from scipy.signal import savgol_filter

'''
'''
def find_peak_near_data(degree_each, xrd_data=[], xrd_step = 0.02):#(这里目前设想再传入一个参数为扫描角度范围)
    j = 0
    low_bdry = round(degree_each) - 1
    while xrd_data[j, 0] != low_bdry:
        j = j + 1
    indice = numpy.where(max(xrd_data[j:j+200, 1]) == xrd_data[j:j+200, 1]) #调用where函数取得I局部极值所对应的indice，并从此获得所对应的2theta
    return xrd_data[indice[0] + j, 0], max(xrd_data[j:j+200, 1])#此处返回值为xrd数据上下一定范围的数据点为了后面找峰
                                                                #需要住的是此处通过where返回的indice是切片后的list的下标，因此返回整个list的话需要再加j

def xrd_peak_data(data=[], spectrum=[], xrd_theta=[20, 90]):#data[]为出入的xrd原始数据，spectrum为从pdf卡片获得的“理论”衍射峰强度，xrd_data用于根据实验的数据角度范围，
                                                            #来判断需要标注几根谱线，而这里正是上面提到的对于某些相，它的前几个峰有的并不明显，需要略过，要加一个强度I判断
    j = 0 #迭代找出小于theta的最后一根光谱
    while spectrum[j] <= xrd_theta[1]:
        j += 1
        if j >= len(spectrum):
            break
    k = len(spectrum[0:j-1]) + 1
    maker = numpy.empty(shape = (k,2)) #此处为xrd实验数据标记的储存numpy_array
    i = 0
    while i < k:
        maker[i, 0], maker[i, 1] = find_peak_near_data(spectrum[i], data) #调用find_peak_near_data获得局部极值即峰，并将其用2列的numpy_array储存
                                                                          #对于此处想在标定时，加上晶面。目前想法是：将spectrum(从pdf卡片获得理论谱线)用字典(或者pandas的dataframe)
                                                                          #尽管pdf卡片的角度可能与自己实验的有些许差距，但我认为这对于标语（110）这样的晶面应该影响不大
        i += 1
    return maker

data_1 = numpy.loadtxt('2.txt')
specific_spectrum_aluminum = numpy.array([38.472, 44.738, 65.133, 78.227,  82.435])
specific_spectrum_silicon = numpy.array([28.442, 47.302, 56.121, 69.130, 76.377, 88.026])
maker_si = xrd_peak_data(data_1, specific_spectrum_silicon)
maker_al = xrd_peak_data(data_1, specific_spectrum_aluminum)

plt.xlabel("Theta",fontsize=13,fontweight='bold')
plt.ylabel("I",fontsize=13,fontweight='bold')
plt.plot(data_1[:, 0], data_1[:, 1])
plt.scatter(maker_al[:,0], maker_al[:, 1] + 20, marker='*', label='Al', s=100)
plt.scatter(maker_si[:,0], maker_si[:, 1] + 20, marker='o', label='Si', s=100)
plt.legend(loc='upper right', title = 'Phases') #图例感觉不太好看需要修改
plt.show()
```

![Output](Figure_1.png)

下面是从PDF文件读取标准spectrum，并将其储存在与其晶面对应成键的dataframe中。

```Python
import numpy
import pandas as pd
import matplotlib.pyplot as plt
name = ['theta', 'cp']
data = pd.read_csv("al.txt", names=name, sep=';') 
print(data.shape,data.cp[0:4],sep='\n')
plt.xlabel("Theta",fontsize=13,fontweight='bold')
plt.ylabel("I",fontsize=13,fontweight='bold')
I = [100,200,100,400]
plt.scatter(data.theta[0:4], I, marker='*',label='Al')
i=0
while i < len(data.theta[0:4]):
    plt.annotate(data.cp[i], xy=(data.theta[i], I[i]), xytext=(-15, 10), textcoords='offset points')
    i += 1
plt.show()
#for i in range(len(data['theta'])):
```

![Output](Figure_2.png)

最后，对xrd相标注的正确性是这个script有用的前提条件つ﹏⊂。。。。。。。。。。。。。。