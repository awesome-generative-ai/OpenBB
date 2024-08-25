# OpenBB平台命令行界面（CLI）

[![Downloads](https://static.pepy.tech/badge/openbb)](https://pepy.tech/project/openbb) 
[![LatestRelease](https://badge.fury.io/py/openbb.svg)](https://github.com/OpenBB-finance/OpenBB) 

| OpenBB致力于通过专注于每个人都能访问的开源基础设施来构建投资研究的未来。 |
| :---------------------------------------------------------------------------------------------------------------------------------------------: |
|              ![OpenBBLogo](https://user-images.githubusercontent.com/25267873/218899768-1f0964b8-326c-4f35-af6f-ea0946ac970b.png)                |
|                                                 请访问我们的网站 [openbb.co](www.openbb.co)                                                 |

## 概览

OpenBB平台命令行界面（CLI）是一个包装了[OpenBB平台](https://docs.openbb.co/platform)的命令行界面。

它提供了一种方便的方式来与OpenBB平台及其扩展进行交互，并且还可以自动化通过OpenBB常规脚本收集数据。

在[这里](https://docs.openbb.co/cli)找到OpenBB平台CLI的最完整文档、示例和使用指南。

## 安装

下面的命令提供了访问OpenBB平台背后的所有可用OpenBB扩展的途径，完整的列表在[这里](https://my.openbb.co/app/platform/extensions)。

```bash
pip install openbb-cli
```

> 注意：在[这里](https://docs.openbb.co/cli/installation)找到最完整的安装提示和技巧。

安装完成后，您可以通过运行以下命令部署OpenBB平台CLI：

```bash
openbb
```

这应该会得到以下输出：
![image](https://github.com/OpenBB-finance/OpenBB/assets/48914296/f606bb6e-fa00-4fc8-bad2-8269bb4fc38e) 

## API密钥

要充分利用OpenBB平台，您需要获取一些API密钥以连接数据提供商。以下是设置它们的3个选项：

1. OpenBB Hub
2. 本地文件

### 1. OpenBB Hub

在[OpenBB Hub](https://my.openbb.co/app/platform/credentials)设置您的密钥，并从<https://my.openbb.co/app/platform/pat>获取您的个人访问令牌以连接您的账户。

> 一旦您登录，通过平台CLI（通过`/account`菜单），您的所有凭证将与OpenBB Hub同步。

### 2. 本地文件

您可以直接在`~/.openbb_platform/user_settings.json`文件中指定密钥。

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
