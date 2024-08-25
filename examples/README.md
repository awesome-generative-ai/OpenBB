# 使用OpenBB平台的Jupyter笔记本示例

这个文件夹是一系列示例笔记本的集合，它们展示了如何开始使用OpenBB平台的一些方法。要运行它们，请确保所选的活跃内核与安装OpenBB的Python虚拟环境相同。

## 目录

### googleColab

这个笔记本在Google Colab环境中安装了OpenBB平台，并提供了以下示例：

- 登录OpenBB Hub
- 设置输出偏好
- 获取期权和公司基本面数据
- 创建条形图可视化

### findSymbols

这个笔记本提供了一个介绍，关于发现、查找和搜索股票代码。

- 搜索
- 查找公司和机构文件
- 按地区和指标筛选股票

### loadHistoricalPriceData

这个笔记本详细介绍了如何使用多种来源收集不同间隔的历史价格数据。

- 使用不同间隔加载数据，并更改来源
- 对股票代码符号学的简要解释
- 重新采样时间序列索引
- 各提供商之间的一些差异，以及比较输出结果

### financialStatements

这组示例介绍了OpenBB平台中的财务报表，并比较了大型零售行业公司的自由现金流收益率。

- 财务报表
- 来自不同来源数据的预期
- 财务属性
- 比率和其他指标

### copperToGoldRatio

这个笔记本解释了如何计算和绘制铜金比率。

- 加载历史最前线月期货价格。
- 从FRED获取10年期美国国债的恒定到期历史系列。
- 执行基本的数据帧操作。
- 使用Plotly图形对象创建图表。

### openbbPlatformAsLLMTools

这个笔记本展示了如何通过利用函数调用，将OpenBB平台用作LLM中的函数。

- 从OpenBB平台函数创建LLM工具
- 将所有OpenBB平台函数转换为LLM工具
- 构建一个可以利用函数调用的基本Langchain代理
- 运行代理

### usdLiquidityIndex

这个笔记本演示了如何查询联邦储备经济数据库并重新创建美元流动性指数。

- 在FRED中搜索系列ID。
- 作为单个调用加载多个系列。
- 从FRED查询中解包数据响应。
- 对数据帧执行算术运算。
- 系列或数据帧的标准化方法。
- 创建图表的简单流程。

### impliedEarningsMove

这个笔记本演示了如何使用免费来源的期权价格计算预期收益变动。

- 获取即将到来的收益日历。
- 获取期权链条数据。
- 获取基础股票的最后价格。
- 找到最接近股票最后价格的看涨和看跌期权的行权价。
- 使用跨式期权的价格计算预期的每日变动。

### streamlit/news

这是一个示例Streamlit仪表板，用于显示来自Biztoc、Benzinga、FMP、Intrinio和Tiingo的新闻头条。

:::warning
至少需要一个API密钥。您可以在[这里](https://rapidapi.com/thma/api/biztoc)获取免费的Biztoc API密钥
:::

要运行，请将文件复制到您的系统，在终端中打开，导航到文件所在的位置，并在激活了`obb` Python环境的情况下，输入：

```
pip install streamlit
pip install openbb-biztoc
streamlit run news.py
```
