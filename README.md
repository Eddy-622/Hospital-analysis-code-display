# Hospital-analysis-code-display

## 以下为朝阳医院相关项目分析代码

```python
# 导入模块
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
plt.rcParams['font.sans-serif'] = ['FangSong']

# 读取数据
df = pd.read_excel(r'朝阳医院2018年销售数据.xlsx',
                   header=None,
                   names=['销售日期','社保卡号','商品编码','商品名称','销量',
                          '应收金额','实收金额'],
                   skiprows=1,
                   dtype=object
                   )

# 观察数据
print(df.head())
print(df.shape)
print(df.info())
print(df.describe())
print('=============================================================')
# 存在缺失值,数据类型不正确

# 删除缺失值
df.dropna(inplace=True)
print(df.count())

# 修改数据类型
df['销量'] = df['销量'].astype(int)
df['应收金额'] = df['应收金额'].astype(float)
df['实收金额'] = df['实收金额'].astype(float)
print(df.dtypes)

# 清洗销售日期 筛去无用信息
df['销售日期'] = df['销售日期'].apply(lambda x:x.split(' ')[0])
print(df['销售日期'].head())

# 转换日期格式
df['销售日期'] = pd.to_datetime(df['销售日期'],errors='coerce')
# 数据中出现错误日期,无法转换,代码报错,添加错误参数跳过错误.
print(df.count())
print(df['销售日期'].head())

# 转换日期格式后出现缺失值,再次清理
df.dropna(inplace=True)
print(df.describe())    # 销量和金额出现负值,表示有退货行为,不符合计算规则

# 通过布尔取值筛除退货订单
df = df[df['销量'] > 0]
print('====================================================')
print(df.describe())

# 通过时间顺序进行排序
df.sort_values(by='销售日期', ignore_index=True, inplace=True) # 排序后索引被打乱，通过ignore_index参数忽略索引

print('========================================================')
# 数据清洗完毕,开始计算KPI
# KPI_I:月均消费次数
# 月均消费次数 = 总消费次数/月份 (当天同一个账户的多次消费按一次消费计算)
# 去重获得总消费次数
kpi_df = df
print('表单内的消费次数:', kpi_df.shape[0])
kpi_df.drop_duplicates(subset=['销售日期','社保卡号'], inplace=True, ignore_index=True)
ti = kpi_df.shape[0]
print('实际消费次数:', ti)

# 统计营业天数,计算大致月份
start = kpi_df.loc[0,'销售日期']
end = kpi_df.loc[ti-1,'销售日期']
print('开始日期:', start)
print('结束日期:', end)
month = (end-start).days/30
print('营业月份:', (end-start).days/30)
print('KPI_I:月均消费次数:', int(ti/month),'次')
print('==================================================')

# KPI2：月均消费金额
# 月均消费金额 = 总消费金额/月份数
# sum函数计算总消费金额
money = np.sum(df['实收金额'])
print('总金额:', money,'元')
KPI_2 = money/month
print('KPI_2:月均消费金额 ',int(KPI_2), '元/月')
print('===================================================')

# KPI_3:客单价
# 客单价 = 总消费金额/总消费次数
KPI_3 = money/ti
print('KPI_3 客单价:%s' % KPI_3)

# KPI4：消费趋势
# 	每日的销售趋势 曲线图
# 	每月的销售趋势 曲线图
#  	药品销售情况（柱状图：销售前十的药品情况表）

# KPI4_I 每日的销售趋势 曲线图  plt.plot
plt.figure('朝阳医院分析图', facecolor='pink',figsize=(6.4*3,4.8*1.5))
plt.tick_params(labelsize=12)
plt.subplot(1,3,1)
plt.title('每日消费趋势图')
plt.xlabel('时间')
plt.ylabel('实收金额')
plt.plot(df['销售日期'], df['实收金额'])
plt.xticks(rotation = 345)

# KPI4_II 每月的销售趋势 曲线图
# 曲线图,x轴月份,y轴消费金额
plt.subplot(1,3,2)
plt.title('每月消费趋势图')
plt.xlabel('月份')
plt.ylabel('消费金额')

# 分组聚合计算每个月的实收金额
month_Money = df.groupby(by=df['销售日期'].dt.month)[['实收金额']].agg('sum')
plt.xticks(ticks= month_Money.index,labels =  ['%s月'%f  for f in range(1,month_Money.index[-1]+1)])
# plt.ticks(ticks(刻度), labels(标签))
plt.plot(month_Money.index,month_Money['实收金额'])
# 113行双[]取值为了将格式转化成这里可以取值的DataFrame格式
for mx, my in zip ( month_Money.index, month_Money['实收金额']):
    plt.text(
        mx, my, '%.1f元'% my,
        ha='center',
        va='center',
        size=12
    )


# KPI4_III 药品销售情况（柱状图：销售前十的药品情况表）
# 按药品种类分组计算各种类的销量 ,sort_values 排序 , 切片取值取前十
# 绘制柱状图,x轴药品种类,y轴销量

plt.subplot(1,3,3)
plt.title('销量前十药品情况表')
plt.xlabel('药品名称')
plt.ylabel('销量')
saves_1 =df.groupby(by=df['商品名称'])[['销量']].agg('sum')
# 排序
saves_1.sort_values(by=['销量'], ascending=False, inplace=True)
# 切片取值
top_ten = saves_1[:10]
print(top_ten)
plt.xticks(rotation=270)
plt.bar(top_ten.index,top_ten['销量'])

# 设置图表显示数值
for tx, ty in zip(top_ten.index,top_ten['销量']):
    plt.text(
        tx, ty+30, '%.i'%ty,
        ha='center',
        va='center',
        size=12
    )

# plt.show()
plt.savefig('朝阳医院销售分析')


```

