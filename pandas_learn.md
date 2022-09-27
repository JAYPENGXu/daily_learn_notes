* pandas中按某一个特征的值取出该值的索引：例如，取出gender特征中值为00 和 0 的index,并将其填充为int型的0

```python
gender_list = train.index[(train['gender'] == '00') | (train['gender'] == '0')].tolist()

for x in gender_list:
    train['gender'][x] = 0
```

* pandas中按值取出特定行

```python
df.loc[df['name'] == 'blue']
df.loc[(df['name'] == 'red') & (df['type'] == 'left')]
```

* pandas获取具有部分字符串匹配条件的行的索引

```python
df.index[df['name'].str.contains('bl')].tolist()
df.loc[df['name'].str.contains('bl')]
df.loc[(df['name'].str.contains('bl')) & (df['type'].str.constains('le'))]
```

* pandas中数据类型转换

```python
df.astype(int)
df['age'].astype(int)
df.round(0).astype(int) #四舍五入为int
#to_numeric将float转换为int
pd.to_numeric(df, downcast='integer')
```

* pandas统计分布

```python
train['age'].value_counts()
train['age'].value_counts().values[0] #输出第一个value的个数
```

* pandas的`groupby`(复合)操作

```python
df = pd.DataFrame({'A': [1, 1, 2, 2],
                  'B': [1, 2, 3, 4],
                   'C': np.random.randn(4)})

df1 = df.groupby('A', as_index=False).agg({'B':'min', 'C':'max'}).rename(columns={'B':'BBB', 'C':'cccc'})


df = df.merge(data,on='A',how='left')
```

