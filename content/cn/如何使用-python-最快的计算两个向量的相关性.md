+++
title = "使用 Python 计算两组数据的相关性，哪种方式更快"
author = ["lkcp"]
date = 2024-02-01T13:59:00+08:00
draft = false
+++

在执行 cpa 时，最耗时的操作就是相关性的计算，所以一个快速的相关性计算方法就尤为重要。使用 python 计算相关性，主要有这么几种方式：


## numpy.corrcoef {#numpy-dot-corrcoef}

我们来看计算它的耗时的情况

```python
import time
import numpy as np
from random import randint

arr0 = np.array([randint(0, 1024) for i in range(655360)])
arr1 = np.array([randint(0, 1024) for i in range(655360)])

total = 0.0
N = 1000

for i in range(N):

    time0 = time.time()

    np.corrcoef(arr0, arr1)

    time1 = time.time()

    total += (time1 - time0)

return total
```

```text
2.963115692138672
```

。


## scipy.stats.pearsonr {#scipy-dot-stats-dot-pearsonr}

```python
from scipy.stats import pearsonr
import time
from random import randint
import numpy as np

arr0 = np.array([randint(0, 1024) for i in range(655360)])
arr1 = np.array([randint(0, 1024) for i in range(655360)])

total = 0.0
N = 1000

for i in range(N):
    time0 = time.time()

    correlation_coefficient, p_value = pearsonr(arr0, arr1)

    time1 = time.time()

    total += (time1-time0)

return total
```

```text
5.543497562408447
```

基本上稳定在 0.078 左右。


## pandas.DataFrame.corr {#pandas-dot-dataframe-dot-corr}

```python
import pandas as pd
import numpy as np
from random import randint
import time

data = {'data1': np.array([randint(0, 1024) for i in range(655360)]),
      'data2': np.array([randint(0, 1024) for i in range(655360)])}

df = pd.DataFrame(data)

total = 0.0
N = 1000

for i in range(N):
    time0 = time.time()
    correlation_matrix = df.corr()
    time1 = time.time()

    total += (time1 - time0)

return total
```

```text
7.046475172042847
```

略慢一些，但是区别不大。


## 总结 {#总结}

综合看来，np.corrcoef 是最快的方式。
