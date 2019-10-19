---
layout: post
title:  "量化金融-Hurst指数"
categories: 计量 data-science
tags:  data-science 
author: blankseraph
---

* content
{:toc}


## Hurst Exponent
Calculates the Hurst exponent of a time series based on Rescaled range (R/S) analysis.  
Reference: https://en.wikipedia.org/wiki/Hurst_exponent 
## Environment  
Python 3.6.2 AMD64  
numpy (1.13.3+mkl)  
pandas (0.20.3)  
## User Guide  
import Hurst  
ts = list(range(50))  
hurst = Hurst.hurst(ts)
## Tips
The input ts has to be object list(n_samples,) or np.array(n_samples,).





















## 赫斯特指数简介
赫斯特指数（英语：Hurst exponent）以英国水文学家哈罗德·赫斯特命名，起初被用来分析水库与河流之间的进出流量，后来被广泛用于各行各业的分形分析。利用Hurst参数可以表征网络流量的自相似性，Hurst参数越大，说明流量的自相似程度就越高，也就是说网络的业务流量在很长的时间内都具有长相关性，这主要是由于网络流量的突发性造成的。现有的文献给出的估计方法主要是两大类：时域法和频域法，其中时域法包括R/S分析法[1]、时间方差图法[2][3]、IDC法，频域法包括Whittle的最大似然估计[4]、小波法[5]等。常用的Hurst估值算法都有不同的适用条件，不能广泛的应用于各种情况，因为每一种算法在时域或者是频域的范围内应用了求和平均的方法，这样就会使得时间序列的高突发可变的细节信息丢失，从而导致出估算结果为负值，增大了估计误差。

## 简单应用
时域法是直接对时间序列进行处理，并用最小二乘法拟合估计出Hurst参数，频域法通过利用FFT对时间序列的谱密度进行估计。时域法及频域法都要求整个观察时间段内全部的时间序列，当时间范围较大时，就需要大量的序列样本和高采样率，同时很难观察到Hurst参数的时变性。同时，对有限长度的时间序列进行Hurst估算，结果虽然可以反映出网络流量局部的突发性，但是由于估值算法容易受到各种因素的干扰而产生误差，并且由于相邻的估算值之间没有数据关联，就不能够体现出突发的渐进性。因此如何估算出无限增长的流量的突发性，同时又能够体现出网络流量变化的全局渐进性，并且还能够体现出局部变化的时变性，这些都需要做进一步的研究。比如，在IDC基础上定义复数取值的赫斯特指数等等。

## 选取实例
本次分析选取了上交所2009年五支股票78日的日收益率数值，进行赫斯特指数的对比分析
![image.png](https://i.loli.net/2019/10/19/JImzgMXVEZYlvNK.png)
首先利用R语言进行描述性分析瞧一瞧这几只股票都长啥样
>install.packages('PerformanceAnalytics')

>install.packages('DAAG')

>install.packages('readr'); library('readr')

>a<-read_csv('hurst.csv')
>names(a)<-c('y','x2','x3','x4','x5')
>summary(a)
### 运行后这样的
![image.png](https://i.loli.net/2019/10/19/MYZKuHEG5xy1WIR.png)
哇这几只股票长得好像啊有没有
尤其是前两支股票其收益数据几乎重叠（怨不得呢，回头看看原始数据本来就很像，连涨跌都好似是同步的）
因为1，2股票相近，3，4股票收益率也相近，那么我就选取1，3进行赫斯特指数对比分析了。。。

### 上代码!
 -*- coding: utf-8 -*-
 Reference: https://en.wikipedia.org/wiki/Hurst_exponent
 python 3.6.2 AMD64
 2018/4/19

 Calculate Hurst exponent based on Rescaled range (R/S) analysis
 How to use (example):
 
 import Hurst 
 ts = list(range(50))
 hurst = Hurst.hurst(ts)
 Tip: ts has to be object list(n_samples,) or np.array(n_samples,)
 __Author__ = "Blank Seraph"

>import numpy as np

>import pandas as pd

>import  csv

>def hurst(ts):

>    N = len(ts)
>    print(N)
>   if N < 20:

>      raise ValueError("Time series is too short! input series ought to have at least 20 samples!")

>   max_k = int(np.floor(N/2))
>  R_S_dict = []
> for k in range(10,max_k+1):

>    R,S = 0,0
>   # split ts into subsets
>  subset_list = [ts[i:i+k] for i in range(0,N,k)]

> if np.mod(N,k)>0:
>    subset_list.pop()
>   #tail = subset_list.pop()
>  #subset_list[-1].extend(tail)
>        # calc mean of every subset
>       mean_list=[np.mean(x) for x in subset_list]

>      for i in range(len(subset_list)):
>         cumsum_list = pd.Series(subset_list[i]-mean_list[i]).cumsum()
>        R += max(cumsum_list)-min(cumsum_list)
>       S += np.std(subset_list[i])

>  R_S_dict.append({"R":R/len(subset_list),"S":S/len(subset_list),"n":k})

>    log_R_S = []
>   log_n = []
>  # print(R_S_dict)

> for i in range(len(R_S_dict)):
>    R_S = (R_S_dict[i]["R"]+np.spacing(1)) / (R_S_dict[i]["S"]+np.spacing(1))
>   log_R_S.append(np.log(R_S))

>  log_n.append(np.log(R_S_dict[i]["n"]))
>   Hurst_exponent = np.polyfit(log_n,log_R_S,1)[0]
>  print(Hurst_exponent)
> return Hurst_exponent

>if __name__ == '__main__':
> ts = list()

>with open('C:/Users/13760/Desktop/hurst.csv', mode='r', encoding='utf-8') as infile:
> read = csv.reader(infile)
>for line in read:
>   ts.append(line[1])

>  # print(ts)

>N = len(ts)
>ts = np.array(ts)
>ts = ts.astype(np.float)

>hurst(ts)
然后我们来看一下这几支股票的hurst指数究竟咋样？（分别修改ts.append(line[1]和line[2])运行
可得第一支股票的hurst指数为0.4386795017068711；第二支股票为0.5293240661196189
；第三支股票为0.6555969221053866；第四支为0.5492369921528519；第五支为0.37444714098902215。
hurst指数的取值范围在 0 和 1 之间（不包括 0 和 1）。当hurst指数=0.5时，该时间序列没有相关性。当hurst指数在0.5~1时，该时间序列有长记忆性；当hurst指数在0~0.5时，该时间序列表现出反持续性，因此它表现出比纯随机更强的波动，由此可以看出第一支和第五支股票的波动性较强，第二支第四支股票波动接近随机，而第三支股票则相对稳健呈现持续性周期性的波动，由于三四支股票收益率整体上相近，所以选取第三支股票比较稳妥。
