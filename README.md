# yyt
import pandas as pd
import matplotlib.pyplot as plt
# 读取逗号分隔的CSV文件
df = pd.read_csv("C:/Users/22629/Desktop/电商数据分析/Online-shopping-sales-data/data/电商网购销售数据_5000条_原始脏数据.csv",sep=",",encoding='utf-8')
raw_df = df.copy()  # 【修正】备份原始数据，用于后续对比验证
print(f"原始数据总行数：{len(raw_df)}")
# 导出为Excel文件
#df.to_excel("C:/Users/22629/Desktop/电商数据分析/Online-shopping-sales-data/data/转换结果.xlsx", index=False, engine="openpyxl")
#print(df.head(10))                                                    #显示前十行数据
#print(df.info())                                                      #显示数据类型
#print(df.isnull().sum())                                              #缺失值统计
df.dropna(subset=['用户ID'],inplace=True)                               #用户ID缺失直接删除该数据
#print(df.duplicated().sum())                                          #显示重复值
df.drop_duplicates(inplace=True)                                       #删除重复项

#统一数据格式
df['用户性别'] = df['用户性别'].str.strip()
df['支付方式'] = df['支付方式'].str.strip().str.replace('银行卡支付','银行卡', regex=False)
df['订单状态'] = df['订单状态'].str.strip().replace('取消','已取消', regex=False).replace('完成','已完成', regex=False)
df['退款状态'] = df['退款状态'].str.strip().replace('未退款','无退款', regex=False)
df.loc[df['退款状态'] == '无','退款状态'] = '无退款'
df['商品名称'] = df['商品名称'].str.strip().str.replace(r'\s+',' ',regex=True)
df['商品类别'] = df['商品类别'].str.strip().str.replace(r'\s+',' ',regex=True)
df['商品ID'] = df['商品ID'].str.upper()                                           #全部统一成大写
df['订单ID'] = df['订单ID'].str.upper()
df['用户ID'] = df['用户ID'].str.upper()
df['省份'] = df['省份'].str.strip().replace('广东','广东省', regex=False).replace('河南','河南省', regex=False).replace('浙江','浙江省', regex=False)
df['城市'] = df['城市'].str.strip().replace({'宁波':'宁波市','宝鸡':'宝鸡市','成都':'成都市','苏州':'苏州市','无锡':'无锡市'}, regex=False)
df.drop_duplicates(subset=['订单ID'], keep='first', inplace=True) # 清洗完成后再去重（【修正】调整顺序，避免格式差异导致去重失效）

df.loc[df['商品ID'] == 'P1002','商品名称'] = 'iPhone 15 Pro'
df.loc[df['商品ID'] == 'P2005','商品名称'] = '机械键盘'
df.loc[df['商品ID'] == 'P4002','商品名称'] = '阿迪达斯卫衣'
df.loc[df['商品ID'] == 'P5001','商品名称'] = '雀巢咖啡'
df.loc[df['商品名称'] == '戴尔 27英寸显示器','商品ID'] = 'P3001'
df.loc[df['商品名称'] == '华为 Mate 70','商品ID'] = 'P1003'
df.loc[df['商品名称'] == 'iPhone 15','商品ID'] = 'P1001'
df.loc[df['商品名称'] == '蒙牛纯牛奶','商品ID'] = 'P5002'
df.loc[df['商品名称'] == '罗技 MX Master 3S','商品ID'] = 'P2004'
df.loc[df['商品名称'] == '机械革命游戏本','商品ID'] = 'P3003'
df.loc[df['商品名称'] == '机械键盘','商品ID'] = 'P2005'
df.loc[df['商品名称'] == '小米手环 9','商品ID'] = 'P2003'
df.loc[df['商品名称'] == 'iPhone 15 Pro','商品ID'] = 'P1002'

#寻找缺失值
df.loc[df['用户性别'].isna(),'用户性别'] = '未知'   #用户性别为空修改为未知
df.loc[df['订单状态'].isna(),'订单状态'] = '未知'   #订单状态为空修改为未知
df.loc[df['退款状态'].isna(),'退款状态'] = '未知'   #退款状态为空修改为未知
df.loc[df['支付方式'].isna(),'支付方式'] = '未知'   #支付方式为空修改为未知
df.loc[df['城市'].isna(),'城市'] = '未知'          #城市为空修改为未知

#统一商品类别
#print(df["退款状态"].unique())                                 #查看商品类别都有什么类型数据
#print(df[["商品ID", "商品名称", "商品类别"]].drop_duplicates())  #显示这三列不同的数据
standard_map = {"P4001": "服饰鞋包","P2001": "数码配件","P2002": "数码配件","P1005": "手机"}
# 根据商品ID替换成标准类别
df["商品类别"] = df["商品ID"].map(standard_map).fillna(df["商品类别"])
#修改数据类型
df['用户性别'] = df['用户性别'].astype('category')
df['支付方式'] = df['支付方式'].astype('category')
df['订单状态'] = df['订单状态'].astype('category')
df['退款状态'] = df['退款状态'].astype('category')
df['商品类别'] = df['商品类别'].astype('category')

#修改时间格式
df['下单时间'] = (
    df['下单时间'].str.strip()
    .str.replace('/', '-', regex=False)
    .str.replace('年', '-', regex=False)
    .str.replace('月', '-', regex=False)
    .str.replace('日', ' ', regex=False)
    .str.replace(r'\s+', ' ', regex=True)
)
df['下单时间'] = pd.to_datetime(df['下单时间'], errors='coerce')
fail_count = df['下单时间'].isna().sum()
print(f"日期解析失败行数：{fail_count}")
df = df[df['下单时间'].notna()].reset_index(drop=True)

#寻找异常值
#print(df[ (df['购买数量']<=0) & (df['商品单价']<=0) & (df['原始金额']<=0)] )    #寻找同时满足购买数量和商品单价和原始金额都小于等于0的数据
#print(df[df['商品单价'] <= 0]['商品单价'])                #寻找商品单价小于等于0或为空的数据
#修改商品单价异常的数据：商品单价 = 原始金额/购买数量
df.loc[(df['商品单价'] <= 0) | (df['商品单价'].isna()),'商品单价'] = df['原始金额']/df['购买数量']
#print(df[df['购买数量'] <= 0]['购买数量'])                #寻找购买数量小于等于0或为空的数据
#修改购买数量异常的数据：购买数量 = 原始金额/商品单价
df.loc[(df['购买数量'] <= 0) | (df['购买数量'].isna()),'购买数量'] = df['原始金额']/df['商品单价']
#print(df[df['原始金额'] <= 0]['原始金额'])                #寻找购买数量小于等于0或为空的数据
#修改购买数量异常的数据：原始金额 = 购买数量*商品单价
df.loc[(df['原始金额'] <= 0) | (df['原始金额'].isna()),'原始金额'] = df['购买数量']*df['商品单价']
df['购买数量'] = df['购买数量'].astype('int64')    #缺失值补全后修改数据类型
#print(df['购买数量'].value_counts().sort_index())  #显示购买数量各有多少条
df = df[(df['购买数量'] < 50)& (df['购买数量'] > 0)]    #排除购买数量过大的异常值

# # —— 验证数据清洗结果 ——
# def has_leading_trailing_space(series):
#     return series.dropna().str.match(r'^\s|\s$').sum()
# # 1.1 检查是否去重
# print("=== 1. 缺失值与去重校验 ===")
# print(f"删除用户ID空值+去重后行数：{len(df)}")
# print(f"用户ID空值数量：{df['用户ID'].isnull().sum()} （应为0）")
# print(f"整行重复数量：{df.duplicated().sum()} （应为0）")
#
# # 2.1 检查是否还有首尾空格
# print(f"用户性别首尾空格数：{has_leading_trailing_space(df['用户性别'])} （应为0）")
# print(f"支付方式首尾空格数：{has_leading_trailing_space(df['支付方式'])} （应为0）")
# print(f"省份首尾空格数：{has_leading_trailing_space(df['省份'])} （应为0）")
#
# # 2.2 枚举值统一检查
# print("\n支付方式枚举：", sorted(df['支付方式'].dropna().unique()))
# # 正常应只有：微信支付、支付宝、花呗、银行卡
# print("订单状态枚举：", sorted(df['订单状态'].dropna().unique()))
# # 正常应只有：已完成、已发货、已取消、待付款、退款中
# print("退款状态枚举：", sorted(df['退款状态'].dropna().unique()))
# # 正常应只有：无退款、退款中、已退款
# print("省份枚举：", sorted(df['省份'].dropna().unique()))
# # 正常都是XX省/XX市（直辖市），无广东、河南、浙江等简称
#
# # 2.3 ID是否全大写
# print(f"\n商品ID含小写字母数量：{df['商品ID'].dropna().str.contains('[a-z]', regex=True).sum()} （应为0）")
# print(f"订单ID含小写字母数量：{df['订单ID'].str.contains('[a-z]', regex=True).sum()} （应为0）")
#
# # 3.1检查每个ID对应名称数量
# id_name_count = df.groupby('商品ID')['商品名称'].nunique()
# print("一个ID对应多个名称的商品：")
# print(id_name_count[id_name_count > 1])
#
# # 3.2检查每个名称对应ID数量
# name_id_count = df.groupby('商品名称')['商品ID'].nunique()
# print("\n一个名称对应多个ID的商品：")
# print(name_id_count[name_id_count > 1])
#
# # 4.1缺失值填充校验
# null_check = df[['用户性别','订单状态','退款状态','支付方式','城市']].isnull().sum()
# print(null_check)
#
# # 5.1检查映射的4个ID是否都变成了标准类别
# for pid in standard_map.keys():
#     cate = df.loc[df['商品ID']==pid, '商品类别'].unique()
#     print(f"商品ID {pid} 对应类别：{cate} （应为 {standard_map[pid]}）")
#
# # 5.2检查同ID是否多类别
# id_cate_count = df.groupby('商品ID')['商品类别'].nunique()
# print("\n一个ID对应多个类别的情况：")
# print(id_cate_count[id_cate_count > 1])
#
# # 6.1日期字段校验
# print(f"原始总行数：{len(raw_df)}")
# print(f"处理后总行数：{len(df)}")
# print(f"删除无效日期行数：{len(raw_df) - len(df) - (len(raw_df) - len(df.dropna(subset=['用户ID'])))}")
# print(f"时间列空值数：{df['下单时间'].isnull().sum()} （应为0）")
# print(f"数据类型：{df['下单时间'].dtype} （应为datetime64[ns]）")
# print(f"日期范围：{df['下单时间'].min()} ~ {df['下单时间'].max()}")
# raw_time = raw_df['下单时间']
# converted = pd.to_datetime(
#     raw_time.str.strip()
#     .str.replace('/', '-', regex=False)
#     .str.replace('年', '-', regex=False)
#     .str.replace('月', '-', regex=False)
#     .str.replace('日', ' ', regex=False),
#     errors='coerce'
# )
# fail_dates = raw_time[converted.isna()].unique()
# print("\n解析失败的原始日期样例：")
# print(fail_dates[:10])
#
# # 7.1 非负非空检查
# print(f"商品单价<=0数量：{(df['商品单价'] <= 0).sum()} （应为0）")
# print(f"购买数量<=0数量：{(df['购买数量'] <= 0).sum()} （应为0）")
# print(f"原始金额<=0数量：{(df['原始金额'] <= 0).sum()} （应为0）")
#
# # 7.2 核心逻辑校验：数量 × 单价 ≈ 金额（允许浮点微小误差）
# diff = abs(df['购买数量'] * df['商品单价'] - df['原始金额'])
# error_rows = (diff > 0.01).sum()
# print(f"数量×单价与金额不一致的行数：{error_rows} （应为0或极个别）")
#
# # 7.3 数量范围检查
# print(f"购买数量最大值：{df['购买数量'].max()} （应<50）")
# print(f"购买数量数据类型：{df['购买数量'].dtype} （应为int64）")
#
# # 8.1数据类型校验
# print(df.dtypes)
#
# #输出数据质量报告
# print("\n" + "="*50)
# print("📊 最终数据质量验收报告")
# print("="*50)
# print(f"总行数：{len(df)}，总列数：{df.shape[1]}")
# print(f"总空值数：{df.isnull().sum().sum()} （理想为0）")
# print(f"订单ID重复数：{df.duplicated(subset=['订单ID']).sum()} （应为0）")
# print(f"日期范围：{df['下单时间'].min().date()} ~ {df['下单时间'].max().date()}")
# print(f"商品SKU数：{df['商品ID'].nunique()}")
# print(f"用户数：{df['用户ID'].nunique()}")
# print(f"总销售额：{df['原始金额'].sum():.2f} 元")
# print(f"平均客单价：{df['原始金额'].mean():.2f} 元")
# print("="*50)

#df.to_csv('C:/Users/22629/Desktop/电商数据分析/Online-shopping-sales-data/data/电商网购销售数据_5000条_处理后数据.csv', encoding='utf_8_sig',index=False)    #导出数据

valid_order = df[(df['订单状态'] == '已完成') & (df['退款状态'] == '无退款')].copy()  #有效订单

#1.2025 年一共完成了多少笔有效订单？
print('一共完成:',valid_order['订单ID'].count(),'笔订单')

#2.2025 年总销售额是多少？
print('总销售额:',valid_order['原始金额'].sum())

#3.平均每笔订单消费多少钱？
a = valid_order['原始金额'].sum() / valid_order['订单ID'].count()
print('平均消费:',round(a,2))

#4.哪个月销售额最高？
valid_order['月份'] = valid_order['下单时间'].dt.month
month_sale = valid_order.groupby('月份')['原始金额'].sum()
print('销售月份最高是:',month_sale.idxmax(),'月',month_sale.max())

#5.哪个月销售额最低？
print('销售月份最低是:',month_sale.idxmin(),'月',month_sale.min())

#6.哪个商品销量最高？
print('销量最高的商品为:',valid_order.groupby('商品名称')['购买数量'].sum().idxmax())

#7.哪个商品销售额最高？
print('销量最高的商品为:',valid_order.groupby('商品名称')['购买数量'].sum().idxmax())

#8.哪个商品类别销售额最高？
print('销售额最高的商品为:', valid_order.groupby('商品类别', observed=True)['原始金额'].sum().idxmax())

#9.哪5个商品贡献了最多销售额？（销售额前五）
product_top5 = (valid_order.groupby('商品名称').agg(销售额=('原始金额','sum'),销量=('购买数量','sum'),订单数=('订单ID','count'))
    .sort_values('销售额', ascending=False).head(5))
print(product_top5)
print('-'*30)

#10.哪个省份销售额最高？
print('销售额最高的省份为:',valid_order.groupby('省份')['原始金额'].sum().sort_values(ascending = False).idxmax())

#11.男用户和女用户谁的销售额更高？
print('销售额最高的性别为:', valid_order.groupby('用户性别', observed=True)['原始金额'].sum().sort_values(ascending=False).idxmax())

#12.哪个省份订单数量最多？
print('订单最多的省份为:',valid_order.groupby('省份')['订单ID'].count().sort_values(ascending = False).idxmax())

#13.为什么这个月销售额比上个月下降？
'''
先列出12个月每月的销售额
print('每月销售额:',month_sale)
以10月与11月做比较（为什么11月销售额比10月销售额下降？）
'''
print(month_sale.index[9],'月',month_sale[10],'元')
print(month_sale.index[10],'月',month_sale[11],'元')
print('销售额下降了:',round((month_sale[10] - month_sale[11]) / month_sale[10] * 100,3),'%')
#查看原因：订单数量是否下降
print('-'*30)
month_sale_order = valid_order.groupby('月份')['订单ID'].count()
print('10月份比11月份订单多:',month_sale_order[10] - month_sale_order[11],'笔')
#具体什么商品类型销量降低
month_sale_mc = valid_order.groupby(['月份','商品类别'], observed=True)['原始金额'].sum().sort_values(ascending=False)
print(month_sale_mc.loc[10])
print(month_sale_mc.loc[11])
#销售额主要差在手机和电脑办公类别

#可视化图表
#  全局样式配置
plt.rcParams['font.sans-serif'] = ['SimHei']  # 解决中文乱码
plt.rcParams['axes.unicode_minus'] = False  # 解决负号显示
plt.rcParams['axes.spines.top'] = False  # 隐藏上边框
plt.rcParams['axes.spines.right'] = False  # 隐藏右边框
plt.rcParams['grid.alpha'] = 0.15  # 网格线透明度
plt.rcParams['figure.dpi'] = 100  # 图表清晰度

# 数据预处理
# 生成月份、季度字段（先确保valid_order是独立副本）
valid_order['月份'] = valid_order['下单时间'].dt.month
valid_order['季度'] = valid_order['下单时间'].dt.quarter

# 汇总数据（分类列统一加observed=True消除警告）
month_sale = valid_order.groupby('月份')['原始金额'].sum().sort_index()
quarter_sale = valid_order.groupby('季度')['原始金额'].sum().sort_index()


#  图表函数库

# 1. 纵向柱状图
def show_bar(title, x_labels, y_values, xlabel, ylabel):
    """
    :param x_labels: x轴类别标签列表
    :param y_values: 对应销售额数值（单位：元）
    """
    y_values = [v / 10000 for v in y_values]  # 统一转万元

    plt.figure(figsize=(10, 5.5))
    plt.title(title, color='#d92323', fontsize=20, pad=15)

    # 蓝色系渐变：数值越高颜色越深
    colors = [plt.cm.Blues(0.45 + i / len(y_values) * 0.45) for i in range(len(y_values))]
    bars = plt.bar(x_labels, y_values, color=colors, width=0.6, label=ylabel)

    # 柱子顶部标注数值
    for bar in bars:
        height = bar.get_height()
        plt.text(bar.get_x() + bar.get_width() / 2, height,
                 f'{height:.1f}万', ha='center', va='bottom', fontsize=11)

    plt.xlabel(xlabel, fontsize=14, labelpad=8)
    plt.ylabel(f'{ylabel}（万元）', fontsize=14, labelpad=8)
    plt.legend(loc='upper right', fontsize=10)
    plt.grid(axis='y')
    plt.ylim(0, max(y_values) * 1.18)  # 顶部留出数值空间
    plt.tight_layout()
    plt.show()


# 2. 横向条形图（修正原函数名拼写错误）
def show_barh(title, values, categories, xlabel, ylabel):
    """
    :param values: 销售额数值（单位：元）
    :param categories: y轴类别标签
    """
    values = [v / 10000 for v in values]  # 转万元

    plt.figure(figsize=(11, 5.5))
    plt.title(title, color='#d92323', fontsize=20, pad=15)

    # 橙色系渐变
    colors = [plt.cm.Oranges(0.45 + i / len(values) * 0.45) for i in range(len(values))]
    bars = plt.barh(categories, values, color=colors, height=0.6, label=xlabel)

    # 条形右侧标注数值
    for bar in bars:
        width = bar.get_width()
        plt.text(width + max(values) * 0.02, bar.get_y() + bar.get_height() / 2,
                 f'{width:.1f}万', va='center', fontsize=11)

    plt.xlabel(f'{xlabel}（万元）', fontsize=14, labelpad=8)
    plt.ylabel(ylabel, fontsize=14, labelpad=8)
    plt.legend(loc='lower right', fontsize=10)
    plt.grid(axis='x')
    plt.xlim(0, max(values) * 1.18)
    plt.tight_layout()
    plt.show()


# 3. 饼图
def show_pie(title, data, labels, start_angle=90):
    data = [v / 10000 for v in data]  # 转万元

    plt.figure(figsize=(9, 9))
    plt.title(title, color='#d92323', fontsize=20, pad=15)

    # 柔和配色方案
    colors = ['#4e79a7', '#f28e2b', '#e15759', '#76b7b2', '#59a14f', '#edc949', '#b07aa1']
    wedges, texts, autotexts = plt.pie(
        data, labels=labels, colors=colors[:len(data)],
        autopct='%1.1f%%', startangle=start_angle,
        pctdistance=0.75, labeldistance=1.1,
        wedgeprops=dict(linewidth=1, edgecolor='white')
    )

    # 调整百分比字体
    for t in autotexts:
        t.set_fontsize(11)
        t.set_color('white')
    for t in texts:
        t.set_fontsize(12)

    plt.tight_layout()
    plt.show()


# 4. 折线图
def show_plot(title, x_labels, y_values, xlabel, ylabel):
    y_values = [v / 10000 for v in y_values]  # 转万元

    plt.figure(figsize=(11, 5.5))
    plt.title(title, color='#d92323', fontsize=20, pad=15)

    plt.plot(x_labels, y_values, color='#1f77b4', linewidth=2.5,
             marker='o', markersize=7, markerfacecolor='white', markeredgewidth=2,
             label=ylabel)

    # 节点标注数值
    for x, y in zip(x_labels, y_values):
        plt.text(x, y + max(y_values) * 0.025,
                 f'{y:.1f}万', ha='center', va='bottom', fontsize=10)

    plt.xticks(x_labels)
    plt.xlabel(xlabel, fontsize=14, labelpad=8)
    plt.ylabel(f'{ylabel}（万元）', fontsize=14, labelpad=8)
    plt.legend(loc='upper left', fontsize=10)
    plt.grid(axis='y')
    plt.ylim(0, max(y_values) * 1.15)
    plt.tight_layout()
    plt.show()


# 各图表生成调用

# 1. 用户消费分布直方图
plt.figure(figsize=(11, 6))
plt.title('2025年用户消费金额分布', color='#d92323', fontsize=20, pad=15)
user_consume = valid_order.groupby('用户ID', observed=True)['原始金额'].sum() / 10000

n, bins, patches = plt.hist(user_consume, bins=60, edgecolor='white', linewidth=0.5, label='消费人群')
# 直方图颜色渐变
for i, patch in enumerate(patches):
    patch.set_facecolor(plt.cm.Blues(i / len(patches)))

# 均值、中位数参考线
mean_val = user_consume.mean()
median_val = user_consume.median()
plt.axvline(mean_val, color='#e15759', linestyle='--', linewidth=2, label=f'均值：{mean_val:.1f}万')
plt.axvline(median_val, color='#f28e2b', linestyle='--', linewidth=2, label=f'中位数：{median_val:.1f}万')

plt.xlabel('消费金额（万元）', fontsize=14, labelpad=8)
plt.ylabel('用户人数', fontsize=14, labelpad=8)
plt.legend(loc='upper right', fontsize=11)
plt.grid(axis='y')
plt.tight_layout()
plt.show()

# 2. TOP5省份销售额（横向条形图）
province_sale = valid_order.groupby('省份', observed=True)['原始金额'].sum().sort_values(ascending=True).tail(5)
show_barh(
    '2025年TOP5销售额省份',
    province_sale.tolist(),
    province_sale.index.tolist(),
    '销售额', '省份'
)

# 3. 商品类别销售额占比（饼图）
cate_sale = valid_order.groupby('商品类别', observed=True)['原始金额'].sum().sort_values(ascending=True)
show_pie(
    '2025年各商品类别销售额占比',
    cate_sale.tolist(),
    cate_sale.index.tolist(),
    start_angle=30
)

# 4. 男女消费比例（饼图）
gender_sale = valid_order[valid_order['用户性别'] != '未知'].groupby('用户性别', observed=True)[
    '原始金额'].sum().sort_values(ascending=True)
show_pie(
    '2025年男女消费比例',
    gender_sale.tolist(),
    gender_sale.index.tolist(),
    start_angle=90
)

# 5. 月度销售额折线图
months = list(range(1, 13))
show_plot(
    '2025年月度销售额',
    months,
    month_sale.reindex(months).fillna(0).tolist(),
    '月度', '销售额'
)

# 6. 季度销售额柱状图（修复原标签不匹配bug）
quarter_labels = ['1季度', '2季度', '3季度', '4季度']
show_bar(
    '2025年季度销售额',
    quarter_labels,
    quarter_sale.reindex([1, 2, 3, 4]).fillna(0).tolist(),
    '季度', '销售额'
)

# 7. 销售额TOP5商品
top5_goods = valid_order.groupby('商品名称', observed=True)['原始金额'].sum().sort_values(ascending=False).head()
show_bar(
    '2025年销售额TOP5商品',
    top5_goods.index.tolist(),
    top5_goods.tolist(),
    '商品名称', '销售额'
)

# 8. 10月-11月销售额对比
compare_month = valid_order[(valid_order['月份'] == 10) | (valid_order['月份'] == 11)].groupby('月份')['原始金额'].sum()
show_bar(
    '2025年10月-11月销售额对比',
    ['10月', '11月'],
    [compare_month.get(10, 0), compare_month.get(11, 0)],
    '月份', '销售额'
)

# 9. 10月vs11月 分品类对比柱状图
oct_sale = valid_order[valid_order['月份'] == 10].groupby('商品类别', observed=True)['原始金额'].sum()
nov_sale = valid_order[valid_order['月份'] == 11].groupby('商品类别', observed=True)['原始金额'].sum()

# 对齐索引，避免错位
cate_list = oct_sale.index.tolist()
oct_values = oct_sale / 10000
nov_values = nov_sale.reindex(cate_list).fillna(0) / 10000

plt.figure(figsize=(12, 6))
plt.title('10月 vs 11月各商品类别销售额对比', color='#d92323', fontsize=18, pad=15)

x = range(len(cate_list))
width = 0.35

plt.bar([i - width / 2 for i in x], oct_values, width=width, color='#1f77b4', label='10月')
plt.bar([i + width / 2 for i in x], nov_values, width=width, color='#ff7f00', label='11月')

# 柱子顶部标注
for i, v in enumerate(oct_values):
    plt.text(i - width / 2, v, f'{v:.1f}万', ha='center', va='bottom', fontsize=9)
for i, v in enumerate(nov_values):
    plt.text(i + width / 2, v, f'{v:.1f}万', ha='center', va='bottom', fontsize=9)

plt.xticks(x, cate_list)
plt.xlabel('商品类别', fontsize=13, labelpad=8)
plt.ylabel('销售额（万元）', fontsize=13, labelpad=8)
plt.legend(loc='upper right', fontsize=11)
plt.grid(axis='y')
plt.ylim(0, max(max(oct_values), max(nov_values)) * 1.2)
plt.tight_layout()
plt.show()

# 10. 品类变化明细表
category_compare = pd.DataFrame({
    '10月销售额(万)': oct_values.round(2),
    '11月销售额(万)': nov_values.round(2)
})
category_compare['销售额变化(万)'] = (category_compare['11月销售额(万)'] - category_compare['10月销售额(万)']).round(2)
category_compare['变化率'] = (category_compare['销售额变化(万)'] / category_compare['10月销售额(万)'] * 100).round(
    2).astype(str) + '%'

print("=" * 50)
print("📊 10-11月商品类别销售额对比")
print("=" * 50)
print(category_compare)

数据分析项目（Python，MySQL）
