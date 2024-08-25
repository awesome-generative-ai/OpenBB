# OpenBB平台

[![Downloads](https://static.pepy.tech/badge/openbb)](https://pepy.tech/project/openbb) 
[![LatestRelease](https://badge.fury.io/py/openbb.svg)](https://github.com/awesome-generative-ai/OpenBB) 

| OpenBB致力于通过专注于每个人都能访问的开源基础设施来构建投资研究的未来。 |
| :---------------------------------------------------------------------------------------------------------------------------------------------: |
|              ![OpenBBLogo](https://user-images.githubusercontent.com/25267873/218899768-1f0964b8-326c-4f35-af6f-ea0946ac970b.png)                |
|                                                 请访问我们的网站 [openbb.co](www.openbb.co)                                                 |

## 概览

OpenBB平台提供了一种方便的方式来访问多个数据提供商提供的原始财务数据。该软件包附带了一个现成的REST API - 这允许任何语言的开发者都能轻松地在OpenBB平台上创建应用程序。

请在 [docs.openbb.co](https://docs.openbb.co/platform) 查看完整文档。

## 安装

下面的命令提供了访问OpenBB平台核心功能的途径。

```bash
pip install openbb
```

这将安装以下数据提供商：

| 扩展名称                | 描述                                                                       | 安装命令                                   | 需要的最低订阅类型 |
| ----------------------- | ------------------------------------------------------------------------- | ------------------------------------------ | ------------------ |
| openbb-benzinga         | [Benzinga](https://www.benzinga.com/apis/en-ca/) 数据连接器                 | pip install openbb-benzinga                 | 付费               |
| openbb-federal-reserve  | [FederalReserve](https://www.federalreserve.gov/data.htm) 数据连接器         | pip install openbb-federal-reserve          | 免费               |
| openbb-fmp              | [FMP](https://site.financialmodelingprep.com/developer/) 数据连接器           | pip install openbb-fmp                      | 免费               |
| openbb-fred             | [FRED](https://fred.stlouisfed.org/) 数据连接器                               | pip install openbb-fred                      | 免费               |
| openbb-intrinio         | [Intrinio](https://intrinio.com/pricing) 数据连接器                           | pip install openbb-intrinio                 | 付费               |
| openbb-oecd             | [OECD](https://data.oecd.org/) 数据连接器                                     | pip install openbb-oecd                     | 免费               |
| openbb-polygon          | [Polygon](https://polygon.io/) 数据连接器                                     | pip install openbb-polygon                  | 免费               |
| openbb-sec              | [SEC](https://www.sec.gov/edgar/sec-api-documentation) 数据连接器             | pip install openbb-sec                      | 免费               |
| openbb-tiingo           | [Tiingo](https://www.tiingo.com/about/pricing) 数据连接器                     | pip install openbb-tiingo                   | 免费               |
| openbb-tradingeconomics | [TradingEconomics](https://tradingeconomics.com/api) 数据连接器               | pip install openbb-tradingeconomics         | 付费               |
| openbb-yahoo-finance    | [Yahoo Finance](https://finance.yahoo.com/) 数据连接器                          | pip install openbb-yfinance                 | 免费               |

要安装扩展核心功能，请指定扩展名称或使用 `all` 来安装所有扩展。

```bash
# 安装单个扩展，例如 openbb-charting 和 yahoo finance
pip install openbb[charting]
pip install openbb-yfinance
```

或者，你可以一次性安装所有扩展。

```bash
pip install openbb[all]
```

> 参考：请访问我们的 [网站](https://docs.openbb.co/platform/installation#installation)。

## Python

```python
>>> from openbb import obb
>>> output = obb.equity.price.historical("AAPL")
>>> df = output.to_dataframe()
>>> df.head()
              open    high     low  ...  change_percent             label  change_over_time
date                                ...
2022-09-19  149.31  154.56  149.10  ...         3.46000  September 19, 22          0.034600
2022-09-20  153.40  158.08  153.08  ...         2.28000  September 20, 22          0.022800
2022-09-21  157.34  158.74  153.60  ...        -2.30000  September 21, 22         -0.023000
2022-09-22  152.38  154.47  150.91  ...         0.23625  September 22, 22          0.002363
2022-09-23  151.19  151.47  148.56  ...        -0.50268  September 23, 22         -0.005027

[5 rows x 12 columns]
```

## API密钥

要充分利用OpenBB平台，您需要获取一些API密钥以连接数据提供商。以下是设置它们的3个选项：

1. OpenBB Hub
2. 运行时
3. 本地文件

### 1. OpenBB Hub

在 [OpenBB Hub](https://my.openbb.co/app/platform/credentials) 设置您的密钥，并从 <https://my.openbb.co/app/platform/pat> 获取您的个人访问令牌以连接您的账户。

```python
>>> from openbb import obb
>>> openbb.account.login(pat="OPENBB_PAT")

>>> # 在OpenBB Hub中持久化更改
>>> obb.account.save()
```

### 2. 运行时

```python
>>> from openbb import obb
>>> obb.user.credentials.fmp_api_key = "REPLACE_ME"
>>> obb.user.credentials.polygon_api_key = "REPLACE_ME"

>>> # 在 ~/.openbb_platform/user_settings.json 中持久化更改
>>> obb.account.save()
```

### 3. 本地文件

您可以直接在 `~/.openbb_platform/user_settings.json` 文件中指定密钥。

用以下模板填充此文件，并将值替换为您的密钥：

```json
{
  "credentials": {
    "fmp_api_key": "REPLACE_ME",
    "polygon_api_key": "REPLACE_ME",
    "benzinga_api_key": "REPLACE_ME",
    "fred_api_key": "REPLACE_ME"
  }
}
```

## REST API

OpenBB平台附带了一个使用FastAPI构建的现成的REST API。使用此命令启动应用程序：

```bash
uvicorn openbb_core.api.rest_api:app --host 0.0.0.0 --port 8000 --reload
```

查看 `openbb-core` [README](https://pypi.org/project/openbb-core/) 以获取更多信息。

## 开发安装

要开发OpenBB平台，您需要具备以下条件：

- Git
- Python 3.8或更高版本
- 安装了 `poetry` 和 `toml` 包的虚拟环境
  - 要安装这些包，请激活您的虚拟环境并运行 `pip install poetry toml`

如何以可编辑模式安装平台？

  1. 激活您的虚拟环境
  2. 导航到 `openbb_platform` 文件夹
  3. 运行 `python dev_install.py` 以在可编辑模式下安装包
