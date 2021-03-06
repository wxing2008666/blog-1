[Algorithm]百度：蓄水池抽样
===========================

### 前言
&emsp;&emsp;曾经面百度被问到过这个问题，对于蓄水池抽样可以有一万种问法，但是万变不离其宗。给一个大小未知的数据，数据的体量统计起来比较耗时，例如我要统计100T的日志文件里有多少条日志。然后要求你随机K条数据，保证每条数据被抽取到的概率都相等。

### Probability and Statistics
&emsp;&emsp;先考虑这么一个问题：给定**N**个数，从中选取**K**个数，第**i**个数被取到的概率是多少？  
&emsp;&emsp;从**N**个数里选取**K**个数的样本大小为 `Cr(N, K) = Pr(N, K) / K! = N! / ((N - K)! * K!)`。  
&emsp;&emsp;如果第**i**个数已经选定，那么从**N - 1**个数里选取**K - 1**个数的样本大小为 `Cr(N - 1, K - 1) = Pr(N - 1, K - 1) / (K - 1)! = (N - 1)! / ((N - 1 - (K - 1))! * (K - 1)!) = (N - 1)! / ((N - K)! * (K - 1)!)`。  
&emsp;&emsp;那么第**i**个数被选到的概率为 `p(i) = Cr(N - 1, K - 1) / Cr(N, K) = K / N`。

### 随机选取一条数据
&emsp;&emsp;先思考简单一点的问题，从大小未知的**N**个数据里随机选取一个数，如果确保每个数被选中的概率都为`1/N`?  
1. 选取第一个数  
2. 后面第**i**个数被保留的概率为**1/i**
3. 后面第**i**个数被丢弃的概率为**(i - 1)/i**  

&emsp;&emsp;那么第一个数最终被保留的概率为`p(1) = 1/2 * 2/3 * 3/4 * ... * (N - 1)/N = 1/N`。  
&emsp;&emsp;第**i**个数最终被保留的概率为**i**被选取，后面全部丢弃的概率，即`p(i) = 1/i * i/(1+i) * (i+1)/(i+2) * ... * (N-1)/N = 1/N`。  
&emsp;&emsp;按照以上步骤选取，不需要知道**N**的大小，遍历一次即可取得随机的值，并且所有数被选取的概率均为**1/N**。

### 随机选取K条数据
&emsp;&emsp;为了保证所有的数据被选取的概率都为**K/N**，可以这么想一下，最后一个数被保留的概率必然为**K/N**，那么第**i**个数被保留的概率为**K/i**。  
1. 选取前K个数，取**i > K**
2. 第**i**个数被保留的概率为**K/i**，这里的保留不表示最终会选取
3. 第**i**个数被丢弃的概率为**(i-K)/i**
4. 第**m(m > i)**个数不替换第**i**个数的概率为`(m-k)/m + k/m * (k-1)/k = (m-1)/m`，意为第**m**个数被丢弃的概率加上第**m**个数被保留但是不替换第**i**个数的概率  

&emsp;&emsp;那么第**i**个数被最终选取的概率为`p(i) = K/i * i/(i+1) * (i+1)/(i+2) * ... * (N-1)/N = K/N`。  
&emsp;&emsp;前**K**个数里第**k**个数被保留的概率为`p(k) = K/(K+1) * (K+1)/(K+2) * ... * (N-1)/N = K/N`，意为后面的数都不会替换第**k**个数。

### 伪代码和例子
**伪代码**  
```c
/*
  S has items to sample, R will contain the result
 */
ReservoirSample(S[1..n], R[1..k])
  // fill the reservoir array
  for i = 1 to k
      R[i] := S[i]

  // replace elements with gradually decreasing probability
  for i = k+1 to n
    j := random(1, i)   // important: inclusive range
    if j <= k
        R[j] := S[i]
```  

[测试代码](https://github.com/linghuazaii/algorithm_testing/blob/master/reservoir.cpp)  
**测试结果**
```
=========== data ===========
92 51 11 69 24 35 17 36 26 98 67 39 83 2 75 56 59 18 32 40 91 86 57 12 14 42 27 62 63 58 30 96 13 68 3 87 71 64 9 22 66 80 20 79 89 81 73 0 94 41 88 28 52 43 16 60 49 7 84 72 29 61 6 45 53 76 8 33 37 15 25 55 70 31 82 47 48 10 90 34 23 74 77 19 85 44 93 54 97 1 38 95 4 21 99 5 78 65 50 46
============================================
=========== random k ===========
77 79 4 97 95 42
============================================

=========== data ===========
92 51 11 69 24 35 17 36 26 98 67 39 83 2 75 56 59 18 32 40 91 86 57 12 14 42 27 62 63 58 30 96 13 68 3 87 71 64 9 22 66 80 20 79 89 81 73 0 94 41 88 28 52 43 16 60 49 7 84 72 29 61 6 45 53 76 8 33 37 15 25 55 70 31 82 47 48 10 90 34 23 74 77 19 85 44 93 54 97 1 38 95 4 21 99 5 78 65 50 46
============================================
=========== random k ===========
60 65 80 86 50 59
============================================

=========== data ===========
92 51 11 69 24 35 17 36 26 98 67 39 83 2 75 56 59 18 32 40 91 86 57 12 14 42 27 62 63 58 30 96 13 68 3 87 71 64 9 22 66 80 20 79 89 81 73 0 94 41 88 28 52 43 16 60 49 7 84 72 29 61 6 45 53 76 8 33 37 15 25 55 70 31 82 47 48 10 90 34 23 74 77 19 85 44 93 54 97 1 38 95 4 21 99 5 78 65 50 46
============================================
=========== random k ===========
92 9 28 65 19 75
============================================
```

### Reference
 - [Reservoir sampling](https://en.wikipedia.org/wiki/Reservoir_sampling)

### 小结
&emsp;&emsp;**GOOD LUCK, HAVE FUN!**
