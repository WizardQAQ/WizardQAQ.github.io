---
title: 使用Python绘制XRD数据图(final)
date: 2019-04-25 22:29:56
tags: Python
---
~~从入门到放弃つ﹏⊂。~~

目前为止头疼的地方有：

- 对于金属间化合物，他们的特征谱线有很多条，一些峰在实验数据中不明显（被噪声所覆盖），虽然可以在maker的函数中加入对I大小的限定，但是由于目前对annotate的晶面（即pdf卡片中的谱线无法进行筛选）不太好标

- 相较于其他相，Al峰强度很高，完全显示Al峰的话其他峰不太明显。但是如果想砍掉一些铝峰，对于铝峰的标注就不能都用一个maker函数生成

- 对多组数据下的xrd，感觉目前画出来并不好看，不同组数据间峰强比例不同（头疼）

这个idea算是帮我复习下Python的用法吧，尽管没有达到理想的效果（完全的script，只需给出xrd实验数据，便可以标注好）。如果牺牲一定自动化，手动给出一些参数，我感觉应该和origin差不多，或者在有三个自定函数下，要比origin方便一丢。

相比上次这次的plot更加面向对象つ﹏⊂（本以为是自定函数里，无法使用plt.annotate,结果却是参数给错了(￣_￣|||)。还有一点是，不需要手动给出pdf卡片可以从文本直接获取（将其储存在字典里），也可以标定对应峰的晶面指数，具体代码见下，另画的图真丑！

```Python
import numpy
import matplotlib.pyplot as plt 
from scipy.signal import savgol_filter
import pandas as pd
'''
'''
def find_peak_near_data(degree_each, xrd_data=[], xrd_step = 0.02):#(这里目前设想再传入一个参数为扫描角度范围)
    j = 0
    low_bdry = round(degree_each) - 1
    while xrd_data[j, 0] != low_bdry:
        j = j + 1
    indice = numpy.where(max(xrd_data[j:j+80, 1]) == xrd_data[j:j+80, 1]) #调用where函数取得I局部极值所对应的indice，并从此获得所对应的2theta
    return xrd_data[indice[0] + j, 0], max(xrd_data[j:j+80, 1])#此处返回值为xrd数据上下一定范围的数据点为了后面找峰
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

def crystal_plane(dataframe_a=[], maker=[]):
    i = 0
    while i < len(maker[:, 0]):
        axes1.annotate(dataframe_a.cp[i],
         xy=(maker[i, 0], maker[i, 1]+300),
         xytext=(-20, 25), 
         textcoords=('offset pixels'))
        i += 1

'''
specific_spectrum_aluminum = numpy.array([38.472, 44.738, 65.133, 78.227,  82.435])
specific_spectrum_silicon = numpy.array([28.442, 47.302, 56.121, 69.130, 76.377, 88.026])
'''
data_1 = numpy.loadtxt('3.txt')
data_2 = numpy.loadtxt('2.txt')
data_3 = numpy.loadtxt('4.txt')

name = ['theta', 'cp']
specific_spectrum_silicon = pd.read_csv("si.txt", names=name, sep=';')
specific_spectrum_aluminum = pd.read_csv("al.txt", names=name, sep=';')

maker_si = xrd_peak_data(data_1, specific_spectrum_silicon.theta)
maker_al = xrd_peak_data(data_1, specific_spectrum_aluminum.theta)

figure1 = plt.figure()

axes1 = figure1.add_axes([0.1, 0.1, 0.8, 0.8])
axes1.set_xlabel("Theta",fontsize=13,fontweight='bold')
axes1.set_ylabel("I",fontsize=13,fontweight='bold')
axes1.plot(data_1[:, 0], data_1[:, 1], label='3#')
axes1.plot(data_2[:, 0], data_2[:, 1]+100, label='2#')
axes1.plot(data_3[:, 0], data_3[:, 1]+200, label='4#')
axes1.scatter(maker_al[:,0], maker_al[:, 1] + 320, marker='*', label='Al', s=100)
axes1.scatter(maker_si[:,0], maker_si[:, 1] + 320, marker='o', label='Si', s=100)
axes1.legend(loc='upper right', title = 'Legend') #图例感觉不太好看需要修改
crystal_plane(specific_spectrum_aluminum, maker_al)
crystal_plane(specific_spectrum_silicon, maker_si)

plt.show()
```

![Output](Figure_1.png)