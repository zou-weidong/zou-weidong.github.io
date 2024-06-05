---
title: "《Python for Data Analysis》第 3 版"
linkTitle: "《Python for Data Analysis》第 3 版"
weight: 2
description: >
  看完语法之后，直接看用Python进行数据分析，这里进行简单整理。
  官方在线书籍：[地址](https://wesmckinney.com/book/)
---

{{% pageinfo %}}
目标是学会用pyton进行数据分析，提取特征工程，为机器学习提供数据支撑，进而做出AIOps、智能运维等应用。
{{% /pageinfo %}}


# 必备的python库

- pandas：数据处理、分析
- numpy：数值计算
- matplotlib：数据可视化
- seaborn：数据可视化
- scipy：科学计算
- scikit-learn：机器学习
- statsmodels：统计分析
- tensorflow：深度学习
- keras：深度学习
- pytorch：深度学习


## NumPy
python中数值计算的基础包之一，提供了多种数值计算函数，包括线性代数、随机数生成、统计函数等。

对于大多数数据分析应用程序，重点关注的领域是：
- 基于数组的快速操作，用于数据整理和清理、子集和过滤、转换以及任何其他类型的计算。
- 常见的数组算法，如 排序，唯一和集合运算
- 高效的描述统计和聚合/汇总数据
- 分组数据操作


NumPy 为一般数值数据处理提供了计算基础，但是更多的是希望使用 pandas 作为大多数统计或者分析的基础，尤其是表格数据。

pandas 还提供了一些更特定于领域的功能，例如时间序列操作，而 NumPy 中没有这些功能。


NumPy 数组创建函数：

- np.array()：从列表或元组创建数组
- np.zeros()：创建指定大小的数组，元素值均为0
- np.ones()：创建指定大小的数组，元素值均为1
- np.empty()：创建指定大小的数组，元素值随机
- np.arange()：创建指定范围的数组


ndarray 常用属性和方法：

- shape：数组形状，表示每个维度大小的元组
- dtype：数组元素类型
- ndim：数组维度
- size：数组元素个数


使用 NumPy 数组进行算术运算：
大小相等的数组之间的任何算术运算都会逐个元素的对应算术运算操作。如：


```python
import numpy as np  


a = np.array([1, 2, 3])  
b = np.array([4, 5, 6])  
c = a + b  
print(c)  

# 输出：[5 7 9]

b > a  
# 输出：[true True True] ,相同大小的数组之间的比较会产生布尔数组

```

### 索引和切片

一维数组的话就和数组类似，下标是索引，切片是切取数组的一部分，肯前否后。如 arr[5:6]

arr[1:4] = 12 ，将 12赋值给数组 arr 的索引 1 到 3 的元素。

注意：**修改了切片中的值之后，原数组也会发生变化。可以使用 arr[5:8].copy() 复制一份副本，然后再赋值。**

在二维数组中，索引和切片的用法类似，只是多了一个维度。如 arr[2,3] 或者 arr[1:3,2:4]。


布尔索引：

布尔索引是指利用数组中布尔值来选择元素的一种方法。

```python
import numpy as np

arr = np.array([1, 2, 3, 4, 5])
bool_arr = arr > 2
new_arr = arr[bool_arr]
print(new_arr)
#输出：[3 4 5]
```

python 中的关键字 and 和 or 不适用于布尔数组，需要使用 & 和 | 来进行布尔运算。


### 伪随机数生成

**numpy.random** 模块提供了多种生成伪随机数的方法。如，可以使用一下方法从标准正态分布中获取 4 * 4 样本数：

```python
import numpy as np

samples = np.random.standard_normal(size = (4, 4))
print(samples)

```

随机数生成器方法：
- permutation()：随机排列数组元素
- shuffle()：随机打乱数组元素
- uniform()：生成均匀分布的随机数
- integers() 从给定的低到高范围内抽取随机整数
- standard_normal()：从均值为0、标准差为1的正态分布中抽取随机数
- normal()：从正态（高斯）分布中抽取样本
- beta()：从beta分布中抽取样本
- binomial()：从二项分布中抽取样本
- chisquare()：从卡方分布中抽取样本
- uniform() 从均匀[0,1]分布中抽取样本


### 通用函数

一元通用函数：
- abs,fabs 逐个元素计算整数、浮点数或者复数的绝对值
- sqrt 计算数组元素的平方根
- exp 计算数组元素的指数
- log 计算数组元素的自然对数
- square 计算数组元素的平方
- ceil 向上取整
- floor 向下取整
- rint 四舍五入
- sign 计算数组元素的符号
- modf 整数部分和小数部分分离


二进制通用函数：
- add 逐个元素相加
- subtract 逐个元素相减
- multiply 逐个元素相乘
- divide 逐个元素相除
- maximum 逐个元素比较大小，取最大值
- minimum 逐个元素比较大小，取最小值
- fmax 逐个元素比较大小，取最大值 


### 数学统计方法

聚合函数，如 sum mean median min max std var

对于多维数组，可以带上参数 axis = ? 来计算给定轴上的数据，从而生成一个少一个维度的数组。

如 arr.mean(axis=1) 表示各行的平均值， arr.sum(axis=0) 表示各列的和。


- sum()：计算数组元素的和
- mean()：计算数组元素的平均值
- median()：计算数组元素的中位数  

- min()：计算数组元素的最小值
- max()：计算数组元素的最大值
- std()：计算数组元素的标准差
- var()：计算数组元素的方差


## Pandas

pandas 是 Python 中最常用的数据处理库，它提供了高级的数据结构和数据分析工具。

pandas 主要有以下特点：
- 表格型数据结构：DataFrame，类似于 Excel 表格
- 丰富的数据处理功能：数据清理、转换、合并、重塑等
- 高效的数学和统计运算：数据聚合、描述统计、回归分析等
- 时间序列分析：时间序列数据分析、时间序列预测等


### DataFrame

DataFrame 是 pandas 中最常用的表格型数据结构，它类似于 Excel 表格，具有行和列索引，可以存储不同类型的数据。

DataFrame 可以通过如下方式创建：

```python
import pandas as pd


# 从字典创建 DataFrame
data = {'name': ['Alice', 'Bob', 'Charlie'], 'age': [25, 30, 35]}
df = pd.DataFrame(data)
print(df)

# 输出：
   name  age
0  Alice   25
1    Bob   30
2  Charlie   35


# 从列表创建 DataFrame
data = [['Alice', 25], ['Bob', 30], ['Charlie', 35]]
df = pd.DataFrame(data, columns=['name', 'age'])
print(df)

# 输出：
   name  age
0  Alice   25
1    Bob   30
2  Charlie   35


# 从 NumPy 数组创建 DataFrame
data = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
df = pd.DataFrame(data, columns=['a', 'b', 'c'])
print(df)

# 输出：
   a  b  c
0  1  2  3
1  4  5  6
2  7  8  9
```

DataFrame 常用属性和方法：

- shape：返回 DataFrame 的行数和列数
- index：返回行索引
- columns：返回列索引
- values：返回 DataFrame 的值
- info()：打印 DataFrame 的信息
- describe()：计算 DataFrame 的汇总统计信息
- head()：返回 DataFrame 的前几行
- tail()：返回 DataFrame 的后几行
- loc[]：通过标签选择数据
- iloc[]：通过位置选择数据
- at[]：通过标签获取单个值
- iat[]：通过位置获取单个值
- sort_values()：按指定列排序
- sort_index()：按索引排序
- apply()：对 DataFrame 进行函数操作



### Series

Series 是 pandas 中另一种重要的数据结构，它类似于一维数组，但可以包含多个数据类型。

Series 可以通过如下方式创建：

```python
import pandas as pd


# 从列表创建 Series
data = [1, 2, 3, 4, 5]
s = pd.Series(data)
print(s)

# 输出：
0    1
1    2
2    3
3    4
4    5
dtype: int64


# 从字典创建 Series
data = {'Alice': 25, 'Bob': 30, 'Charlie': 35}
s = pd.Series(data)
print(s)

# 输出：
Alice      25
Bob        30
Charlie    35
dtype: int64
```

Series 常用属性和方法：

- shape：返回 Series 的长度
- index：返回索引
- values：返回 Series 的值
- dtype：返回 Series 的数据类型
- nunique()：计算 Series 中唯一值的个数
- unique()：返回 Series 中唯一值的数组
- sort_values()：按值排序
- sort_index()：按索引排序
- apply()：对 Series 进行函数操作
- isin()：判断值是否在 Series 中
- dropna()：删除缺失值
- fillna()：填充缺失值
- replace()：替换指定值
- map()：对值进行映射
- astype()：转换数据类型


### 重建索引 reindex

reindex() 方法可以重新设置 DataFrame 的索引。

对于时间序列等有序数据，可以在重新索引时对数据进行填充、对齐等操作。

- ffill()：用前一个值填充缺失值
- bfill()：用后一个值填充缺失值

**reindex** 函数参数:

- labels：新的索引标签
- method：填充方式，ffill 前一个值，bfill 后一个值
- axis：0 行索引，1 列索引
- columns：指定列名，默认是所有列
- limit: 最大填充次数，默认是无穷大
- level：多级索引的层级，否则选择子集
- copy : 是否复制数据，默认是 True


### 删除index

drop() 方法可以删除指定列或行。

> pd.Series(np.arange(5.),index=['a','b','c','d','e']).drop('c')

输出：

```
a    0.0
b    1.0  
d    3.0  
e    4.0  
dtype: float64
```

### 时间序列处理

pandas 提供了一些时间序列处理的函数，如日期的解析、转换等。

- to_datetime()：将字符串转换为日期格式
- date_range()：生成日期序列
- resample()：重采样数据  

```python
import pandas as pd

# 解析日期字符串
dates = ['2019-01-01', '2019-01-02', '2019-01-03']
df = pd.DataFrame({'value': [1, 2, 3]}, index=pd.to_datetime(dates))
print(df)


# 重采样数据
df = pd.DataFrame({'value': [1, 2, 3, 4, 5, 6, 7, 8]}, index=pd.date_range('2019-01-01', '2019-01-08'))
df = df.resample('D').sum()
print(df)
```

### 读取和写入文件

pandas 具有许多用于将表格数据读取为 DataFrame 对象的函数，如 pandas.read_svc 就是这本书中最常用的函数之一


将文本数据转换为 DataFrame ：
- read_csv()：读取 CSV 文件
- read_excel()：读取 Excel 文件
- read_json()：读取 JSON 文件
- read_html()：读取 HTML 文件
- read_sql()：读取 SQL 数据库
- read_fwf()：读取固定宽度文件
- read_clipboard()：读取剪贴板数据


> pd.read_csv('data.csv',keep_default_na=False),na_values=['NA']


read_csv() 函数参数：

- path ：文件路径
- header：行索引，默认是第一行
- names：列名
- index_col：列索引，默认是第一列
- skiprows：跳过的行数
- na_values：缺失值列表
- data_parser：自定义数据解析器
- skip_footer：跳过的行数
- keep_default_na：是否保留缺失值
- thousands：千位分隔符


分段读取文本文件，先将 pandas 设置的更紧凑一些：

> pd.options.display.max_rows = 10

省略号...表示DataFrame中间的行已被省略。
如果想读取少量的行，避免读取整个文件，可以指定 nrows 参数。

> pd.read_csv('data.csv',nrows=10)

要分块读取文件，指定 chunksize 参数：

> for chunk in pd.read_csv('data.csv',chunksize=100):


写入文本：

- to_csv()：写入 CSV 文件
- to_excel()：写入 Excel 文件
- to_json()：写入 JSON 文件


可以指定分隔符，为空的时候标记值，以及禁用行和列的标签写入。或者自定义部分列（columns=['a','b']）的写入格式。

> data.to_csv('data.csv',sep='|', na_rep='Null',index=False, header=False)


二进制数据格式：
- pickle 模块可以用于保存和加载 Python 对象到磁盘。使用 to_pickle() 和 read_pickle() 方法可以将 DataFrame 保存为二进制文件。


Excel 文件读取：

> excel = pd.ExcelFile('file.xlsx')  # 打开 Excel 文件
> df = pd.read_excel('file.xlsx', sheet_name='Sheet1')  # 读取指定工作表

显示工作表：
> excel.sheet_names  # 显示工作表名称

读取多个工作表：  

> dfs = {sheet_name: pd.read_excel('file.xlsx', sheet_name=sheet_name) for sheet_name in excel.sheet_names}  # 读取所有工作表

将数据读入 DataFrame中：
> excel.parse(shell_name="Sheet1")

- index_col：指定索引列，默认是第一列。


要想将 pandas 数据写入 Excel ，需要先创建一个 ExcelWriter 对象，然后调用 to_excel() 方法。

```python
import pandas as pd

# 创建 DataFrame
df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})

# 写入 Excel 文件 
writer = pd.ExcelWriter('file.xlsx')  # 创建 ExcelWriter 对象
df.to_excel(writer, sheet_name='Sheet1')  # 写入数据
writer.save()  # 保存文件

writer.close()  # 关闭文件
```


HDF5 文件格式：

先安装 PyTables 库：

> pip install tables


HDFStore 类可以将 pandas DataFrame 保存为 HDF5 文件。

```python
import pandas as pd
import tables

# 创建 DataFrame
df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})

# 写入 HDF5 文件
with pd.HDFStore('file.h5') as store:
    store['df'] = df  # 保存 DataFrame

# 读取 HDF5 文件
with pd.HDFStore('file.h5') as store:
    df = store['df']  # 读取 DataFrame
```

HDFStore 类常用方法：

- put()：保存 DataFrame
- get()：读取 DataFrame
- select()：选择 DataFrame 的子集
- remove()：删除 DataFrame
- keys()：返回 DataFrame 的键列表


### 