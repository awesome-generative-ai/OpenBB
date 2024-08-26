
# 贡献到 OpenBB 平台

<!-- markdownlint-disable MD033 MD024 -->

以下是代码中的英文文案的中文翻译，以及包含翻译的完整代码：

- [贡献至OpenBB平台](#贡献至openbb平台)
  - [引言](#引言)
    - [OpenBB平台快速一览](#openbb平台快速一览)
      - [什么是标准化框架？](#什么是标准化框架)
        - [标准化的注意事项](#标准化的注意事项)
        - [标准QueryParams示例](#标准queryparams示例)
        - [标准数据示例](#标准数据示例)
      - [什么是扩展？](#什么是扩展)
        - [扩展的类型](#扩展的类型)
  - [依赖管理](#依赖管理)
    - [高级概述](#高级概述)
    - [核心依赖管理](#核心依赖管理)
      - [安装](#安装)
      - [使用Poetry](#使用poetry)
    - [核心与扩展](#核心与扩展)
      - [安装](#安装)
      - [使用Poetry管理依赖](#使用poetry管理依赖)
  - [开发者指南](#开发者指南)
    - [对开发者的期望](#对开发者的期望)
    - [如何构建OpenBB扩展？](#如何构建openbb扩展)
    - [构建扩展：最佳实践](#构建扩展最佳实践)
    - [如何添加新的数据点？](#如何添加新的数据点)
      - [确定要添加的数据类型](#确定要添加的数据类型)
      - [检查标准模型是否存在](#检查标准模型是否存在)
        - [创建查询参数模型](#创建查询参数模型)
        - [创建数据输出模型](#创建数据输出模型)
        - [构建Fetcher](#构建fetcher)
      - [使提供商可见](#使提供商可见)
    - [如何添加自定义数据源？](#如何添加自定义数据源)
      - [OpenBB平台命令](#openbb平台命令)
    - [架构考虑](#架构考虑)
      - [重要类](#重要类)
      - [导入语句](#导入语句)
      - [TET模式](#tet模式)
      - [错误](#错误)
      - [数据处理命令](#数据处理命令)
        - [Python接口](#python接口)
        - [API接口](#api接口)
  - [贡献者指南](#贡献者指南)
    - [对贡献者的期望](#对贡献者的期望)
    - [质量保证](#质量保证)
      - [单元测试](#单元测试)
      - [集成测试](#集成测试)
      - [导入时间](#导入时间)
    - [分享你的扩展](#分享你的扩展)
      - [将你的扩展发布到PyPI](#将你的扩展发布到pypi)
        - [设置](#设置)
        - [版本上线](#版本上线)
        - [发布](#发布)
    - [管理扩展](#管理扩展)
      - [将扩展作为依赖项添加](#将扩展作为依赖项添加)
    - [编写代码并提交](#编写代码并提交)
      - [如何创建PR？](#如何创建pr)
        - [分支命名约定](#分支命名约定)

## 介绍

本文档提供了贡献到 OpenBB 平台的指南。在本文档中，我们将区分两种类型的贡献者：开发者和贡献者。

1. **开发者**: 为 OpenBB 平台构建新功能或扩展，或利用 OpenBB 平台的人。
2. **贡献者**: 通过开启一个[拉取请求](#getting_started-create-a-pr)来贡献现有代码库，从而回馈社区。

**为什么这种区分很重要？**

OpenBB 平台被设计为基础开发的基础。我们预计它将有广泛的创造性用例。有些用例可能非常特定或注重细节，解决可能不一定适合 OpenBB 平台 Github 存储库的特定问题。这完全是可以接受的，甚至是鼓励的。本文档提供了一个全面的指南，介绍如何构建您自己的扩展，添加新的数据点等。

本文档中定义的**开发者**角色可以被视为基础角色。开发者是使用 OpenBB 平台原样或在其上构建的人。

相反，**贡献者**角色指的是那些通过直接添加到 OpenBB 平台或通过扩展[扩展存储库](/openbb_platform/extensions/)来增强 OpenBB 平台代码库的人。贡献者愿意多走一英里，花费额外的时间进行质量保证、测试或与 OpenBB 开发团队合作，以确保遵守标准，从而回馈社区。

### OpenBB 平台快速一览

OpenBB 平台由开源社区构建，其特点是核心和扩展。核心处理数据集成和标准化，而扩展启用定制和高级功能。OpenBB 平台旨在既可以通过 Python 接口也可以通过 REST API 使用。

REST API 建立在 FastAPI 之上，可以通过从根目录运行以下命令来启动：

```bash
uvicorn openbb_platform.core.openbb_core.api.rest_api:app --host 0.0.0.0 --port 8000 --reload
```

我们向用户提供的 Python 接口是 `openbb` Python 包。

您在这个包中找到的代码是由脚本生成的，它只是 `openbb-core` 和任何已安装扩展的包装器。

当用户运行 `import openbb`, `from openbb import obb` 或其他变体时，会触发生成打包代码的脚本。它检测环境中是否安装了新扩展，并相应地重建打包代码。如果未找到新扩展，则只使用当前打包版本。

当您正在开发时，可能想要手动触发包重建。

您可以这样做：

```python
python -c "import openbb; openbb.build()"
```

Python 接口可以这样导入：

```python
from openbb import obb
```

本文档将带您了解两种类型的贡献：

1. 构建自定义扩展
2. 直接贡献到 OpenBB 平台

在继续之前，请查看 OpenBB 平台架构的高级视图。我们将在本文档中逐一介绍每个部分。

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://github.com/OpenBB-finance/OpenBB/assets/74266147/c9a5a92a-28b6-4257-aefc-deaebe635c6a">
  <img alt="OpenBB Platform High-Level Architecture" src="https://github.com/OpenBB-finance/OpenBB/assets/74266147/c9a5a92a-28b6-4257-aefc-deaebe635c6a">
</picture>

#### 什么是标准化框架？

标准化框架是一套工具和指南，使用户能够以一致的方式跨多个提供商查询和获取数据。

每个数据模型应该继承自 OpenBB 平台内已经定义的[标准数据](platform/provider/openbb_core/provider/standard_models)模型。所有标准模型都由 OpenBB 团队创建和维护。

使用这些模型将解锁仅对标准化数据可用的一系列优势，即：

- 可以以标准化的方式查询和输出数据。
- 可以期望遵循标准化的扩展能够即插即用。
- 可以期望 API 返回的数据具有透明定义的模式。
- 可以期望一致的数据类型和验证。
- 将与使用相同标准模型的其他提供商无缝协作。

标准模型定义在 `/OpenBBTerminal/openbb_platform/platform/core/provider/openbb_core/provider/standard_models/` 目录下。

它们定义了 [`QueryParams`](platform/provider/openbb_core/provider/abstract/query_params.py) 和 [`Data`](platform/provider/openbb_core/provider/abstract/data.py) 模型，这些模型用于查询和输出数据。它们是 pydantic 的，您可以利用所有 pydantic 特性，如验证器。

##### 标准化注意事项

标准化框架是一个非常强大的工具，但它有一些您应该注意的注意事项：

- 我们标准化在两个或更多提供商之间共享的字段。如果有第三个提供商不共享相同的字段，我们将将其声明为 `Optional` 字段。
- 当将列名从特定于提供商的模型映射到标准模型时，CamelCase 到 snake_case 的转换是自动完成的。如果列名不相同，您需要手动映射它们。（例如 `o` -> `open`）
- 标准模型由 OpenBB 团队创建和维护。如果您想向标准模型添加一个新字段，您需要向 OpenBB 平台开启一个 PR。

##### 标准 QueryParams 示例

```python
class EquityHistoricalQueryParams(QueryParams):
    """股票历史尾盘查询. Equity Historical end of day Query."""
    symbol: str = Field(description=QUERY_DESCRIPTIONS.get("symbol", ""))
    start_date: Optional[date] = Field(
        description=QUERY_DESCRIPTIONS.get("start_date", ""), default=None
    )
    end_date: Optional[date] = Field(
        description=QUERY_DESCRIPTIONS.get("end_date", ""), default=None
    )
```

`QueryParams` 是一个抽象类，它仅仅告诉我们我们正在处理查询参数。

OpenBB 平台动态地知道标准模型在继承树中的起始位置，所以你不需要担心它。

##### 标准数据示例

```python
class EquityHistoricalData(Data):
    """股票历史尾盘价格数据. Equity Historical end of day price Data."""

    date: datetime = Field(description=DATA_DESCRIPTIONS.get("date", ""))
    open: PositiveFloat = Field(description=DATA_DESCRIPTIONS.get("open", ""))
    high: PositiveFloat = Field(description=DATA_DESCRIPTIONS.get("high", ""))
    low: PositiveFloat = Field(description=DATA_DESCRIPTIONS.get("low", ""))
    close: PositiveFloat = Field(description=DATA_DESCRIPTIONS.get("close", ""))
    volume: float = Field(description=DATA_DESCRIPTIONS.get("volume", ""))
    vwap: Optional[PositiveFloat] = Field(description=DATA_DESCRIPTIONS.get("vwap", ""), default=None)
```

`Data` 类是一个抽象类，告诉我们预期的输出数据。在这里我们可以看到 `vwap` 字段是 `Optional`。这是因为并非所有提供商都共享此字段，而它在两个或更多提供商之间是共享的。

#### 什么是扩展？

扩展为 OpenBB 平台增加了功能。它可以是一个新的数据源、一个新的命令、一个新的可视化等。

##### 扩展的类型

我们主要有 3 种类型的扩展：

1. OpenBB 扩展 - 由 OpenBB 团队构建和维护（例如 `openbb-equity`）
2. 社区扩展 - 由任何人构建，主要由 OpenBB 维护（例如 `openbb-yfinance`）
3. 独立扩展 - 由任何人独立构建和维护

如果您的扩展质量很高，并且您认为它将是一个很好的社区扩展，您可以向 OpenBB 平台存储库开启一个 PR，我们将对其进行审查。

我们鼓励通过将独立扩展发布到 PyPI 与社区分享。

## 依赖管理

### 高级概述

- **提供商（Provider）**: 没有依赖其他 `openbb` 包的基础包。
- **核心（Core）**: 依赖于提供商，作为主要的基础设施包。
- **扩展（Extensions）**: 利用核心的基础设施的实用包。每个扩展都是它自己的包。
- **提供商（Providers）**: 扩展到不同提供商的实用包，每个提供商都是它自己的包。

### 核心依赖管理

#### 安装

- **pip**: `pip install -e OpenBBTerminal/openbb_platform/platform/core`
- **poetry**: `poetry install OpenBBTerminal/openbb_platform/platform/core`

#### 使用 Poetry

在调整依赖之前，请确保您处于一个干净的 conda 环境中。

- **添加依赖**: `poetry add <my-dependency>`
- **更新依赖**:
  - 全部: `poetry update`
  - 特定: `poetry update <my-dependency>`
- **移除依赖**: `poetry remove <my-dependency>`

### 核心和扩展

#### 安装

对于开发设置，请使用提供的脚本安装所有扩展及其依赖项：

- `python dev_install.py [-e|--extras]`

> **注意**: 如果正在开发扩展，请避免安装所有扩展以防止不必要的开销。

#### 使用 Poetry 管理依赖

- **添加平台扩展**: `poetry add openbb-extension-name [--dev]`
- **解决冲突**:如果 Poetry 通知，调整 `pyproject.toml` 中的版本。
- **锁定依赖**：`poetry lock`
- **更新平台**：`poetry update openbb-platform`
- **文档**：维护 `pyproject.toml` 和 `poetry.lock` 以清晰记录依赖。

## 开发者指南

### 对开发者的期望

1. 使用案例：
   - 确保您的扩展或功能与应用程序的更广泛目标一致。
   - 理解 OpenBB 平台被设计为基础；以补充而不是与其核心功能冲突的方式构建。

2. 文档：
   - 为任何新功能或扩展提供清晰全面的文档。

3. 代码质量：
   - 遵守 OpenBB 平台的编码标准和约定。
   - 确保您的代码可维护、组织良好，并在必要时进行注释。

4. 测试：
   - 对任何新功能或扩展进行彻底测试，确保其按预期工作。

5. 性能：
   - 确保您的扩展或功能不会对 OpenBB 平台的性能产生不利影响。
   - 优化可扩展性，特别是如果您预计您的功能需求量大。

6. 协作：
   - 与 OpenBB 社区互动，收集对您的开发的反馈。

### 如何构建 OpenBB 扩展？

我们有一个 Cookiecutter 模板，可以帮助您开始。它作为您扩展开发的起点，让您可以专注于数据而不是样板代码。

请参阅 [Cookiecutter 模板](https://github.com/OpenBB-finance/openbb-cookiecutter) 并按照那里的说明进行操作。

本文档将指导您完成向 OpenBB 平台添加新扩展的步骤。

高级步骤是：

- 生成扩展结构
- 安装您的依赖项
- 安装您的新包
- 使用您的扩展（无论是从 Python 还是 API 接口）
- 对您的扩展进行 QA
- 与社区分享您的扩展

### 构建扩展：最佳实践

1. **审查平台依赖**：在添加任何依赖之前，请确保它与平台的现有依赖一致。
2. **使用宽松版本控制**：如果可能，指定一个范围以保持兼容性。例如，`>=1.4,<1.5`。
3. **测试**：使用平台的核心测试您的扩展，以避免冲突。推荐进行单元和集成测试。
4. **记录依赖**：使用 `pyproject.toml` 和 `poetry.lock` 清晰、及时地记录。

### 如何添加新的数据点？

在本节中，我们将向 OpenBB 平台添加一个新的数据点。我们将添加一个新的提供商，使用现有的[标准数据](platform/provider/openbb_core/provider/standard_models)模型。

#### 确定您要添加的数据类型

在这个例子中，我们将添加 OHLC 股票数据，该数据被 `obb.equity.price.historical` 命令使用。

请注意，如果没有为您的数据存在命令，我们需要在正确的路由器下添加一个。
每个路由器都归类在不同的扩展下（股票、货币、加密货币等）。

#### 检查标准模型是否存在

鉴于已经有 OHLCV 股票数据的端点，我们可以检查标准是否存在。

在这种情况下，它是 `EquityHistorical`，可以在 `/OpenBBTerminal/openbb_platform/platform/core/provider/openbb_core/provider/standard_models/equity_historical` 中找到。

如果标准模型不存在：

- 您将不需要在下一步中继承它。
- 您的所有提供商查询参数将在 Python 接口中的 `**kwargs` 下。
- 它可能无法即插即用地与其他遵循标准化的扩展一起工作，例如 `charting` 扩展

##### 创建查询参数模型

查询参数是传递给 API 端点以进行请求的参数。

对于 `EquityHistorical` 示例，它看起来像下面这样：

```python

class <ProviderName>EquityHistoricalQueryParams(EquityHistoricalQueryParams):
    """<ProviderName> 股票历史查询。

    来源：<url id="" type="url" status="" title="" wc="">https://www.<provider_name>.co/documentation/</url>  
    """

    # 如果有任何特定于提供商的查询参数

```

##### 创建数据输出模型

数据输出是 API 端点返回的数据。
对于 `EquityHistorical` 示例，它看起来像下面这样：

```python

class <ProviderName>EquityHistoricalData(EquityHistoricalData):
    """<ProviderName> 股票历史数据。

    来源：<url id="" type="url" status="" title="" wc="">https://www.<provider_name>.co/documentation/</url>  
    """

    # 如果有任何特定于提供商的数据输出字段

```

> 请注意，由于 `EquityHistoricalData` 继承自 pydantic 的 `BaseModel`，我们可以利用验证器对输出模型执行额外的检查。一个很好的例子是将字符串日期转换为 datetime 对象。

##### 构建 Fetcher

`Fetcher` 类负责向 API 端点发出请求并提供输出。

它将接收查询参数，并在利用 pydantic 模型模式的同时返回输出。

对于 `EquityHistorical` 示例，它看起来像下面这样：

```python
class <ProviderName>EquityHistoricalFetcher(
    Fetcher[
        <ProviderName>EquityHistoricalQueryParams,
        List[<ProviderName>EquityHistoricalData],
    ]
):
    """转换查询，提取和转换数据."""

    @staticmethod
    def transform_query(params: Dict[str, Any]) -> <ProviderName>EquityHistoricalQueryParams:
        """转换查询参数."""

        return <ProviderName>EquityHistoricalQueryParams(**transformed_params)

    @staticmethod
    def extract_data(
        query: <ProviderName>EquityHistoricalQueryParams,
        credentials: Optional[Dict[str, str]],
        **kwargs: Any,
    ) -> dict:
        """从端点返回原始数据."""

        obtained_data = my_request(query, credentials, **kwargs)

        return obtained_data

    @staticmethod
    def transform_data(
        query: <ProviderName>EquityHistoricalQueryParams,
        data: dict,
        **kwargs: Any,
    ) -> List[<ProviderName>EquityHistoricalData]:
        """将数据转换为标准格式."""

        return [<ProviderName>EquityHistoricalData.model_validate(d) for d in data]
```

> 确保您在构建 `Fetcher` 时遵循 TET 模式 - **转换，提取，转换**。在[这里](#the-tet-pattern)了解更多。

默认情况下，每个 `Provider` 上声明的凭据都是必需的。这意味着在执行查询之前，我们会检查所有凭据是否都在场，如果没有就会引发异常。如果您想让某个 fetcher 的凭据变为可选，即使在 `Provider` 上声明了它们，您可以将 `require_credentials=False` 添加到 `Fetcher` 类中。请参阅以下示例：

```python
class <ProviderName>EquityHistoricalFetcher(
    Fetcher[
        <ProviderName>EquityHistoricalQueryParams,
        List[<ProviderName>EquityHistoricalData],
    ]
):
    """转换查询，提取和转换数据."""

    require_credentials = False

    ...
```

#### 使提供商可见

为了使新提供商对 OpenBB 平台可见，您需要将其添加到 `providers/<provider_name>/openbb_<provider_name>/` 文件夹的 `__init__.py` 文件中。

```python
"""<Provider Name> 提供商模块."""
from openbb_core.provider.abstract.provider import Provider

from openbb_<provider_name>.models.equity_historical import <ProviderName>EquityHistoricalFetcher

<provider_name>_provider = Provider(
    name="<provider_name>",
    website="<URL to the provider website>",
    description="提供商描述在这里",
    credentials=["api_key"],
    fetcher_dict={
        "EquityHistorical": <ProviderName>EquityHistoricalFetcher,
    },
)
```

如果提供商不需要任何凭据，您可以删除该参数。另一方面，如果它需要超过 2 个项目进行身份验证，您可以将所有所需项目添加到 `credentials` 列表中。

在 `openbb_platform/providers/<provider_name>` 上运行 `pip install .` 后，您的提供商应该已经准备好使用了，无论是从 Python 接口还是 API。

### 如何添加自定义数据源？

您将从 CSV 文件、本地数据库或 API 端点获取您的数据。

如果您不想或不需要参与数据标准化框架，您可以选择将所有逻辑直接放在路由器文件中。这通常适用于您从本地 CSV 文件返回自定义数据的情况，或类似情况。请记住，我们还提供 REST API，您不应将不可序列化的对象作为响应发送（例如 pandas dataframe）。

也就是说，我们强烈建议遵循标准化框架，因为这将使您的长期工作更轻松，并解锁仅对标准化数据可用的一组功能。

当标准化时，所有数据都使用两种不同的 pydantic 模型定义：

1. 定义[查询参数](platform/provider/openbb_core/provider/abstract/query_params.py)模型。
2. 定义结果[数据模式](platform/provider/openbb_core/provider/abstract/data.py)模型。

> 模型可以完全是自定义的，也可以从 OpenBB 标准化模型继承。
> 它们强化了安全且一致的数据结构、验证和类型检查。

我们称这为 ***了解您的数据*** 原则。

在定义了这两个模型之后，您需要定义一个 `Fetcher` 类，其中包含三个方法：

1. `transform_query` - 转换查询参数以适应 API 端点的格式。
2. `extract_data` - 向 API 端点发出请求并返回原始数据。
3. `transform_data` - 将原始数据转换为定义的数据模型。

> 注意，`Fetcher` 应该从 [`Fetcher`](platform/provider/openbb_core/provider/abstract/fetcher.py) 类继承，这是一个通用类，接收查询参数和数据模型作为类型参数。

在完成您的模型后，您需要使它们对 Openbb 平台可见。这通过将 `Fetcher` 添加到 `<your_package_name>/<your_module_name>` 文件夹的 `__init__.py` 文件中作为 [`Provider`](platform/provider/openbb_core/provider/abstract/provider.py) 的一部分来完成。

任何使用您刚刚定义的 `Fetcher` 类的命令，都将在后台调用 `transform_query`、`extract_data` 和 `transform_data` 方法，以获取数据并将其输出给最终用户。

如果您不确定什么是命令，以及为什么它甚至使用 `Fetcher` 类，请继续关注！

#### OpenBB 平台命令

OpenBB 平台将使您能够以非常简单的方式查询和输出您的数据。

> 任何平台端点都将同时从 Python 接口和 API 可用。

平台上的命令定义遵循 [FastAPI](https://fastapi.tiangolo.com/) 约定，这意味着您将创建 **端点**。

Cookiecutter 模板为您生成了一个 `router.py` 文件，其中包含一组示例供您参考，即：

- 执行一个简单的 `GET` 和 `POST` 请求 - 不用担心任何自定义数据定义。
- 使用自定义数据定义，以便您以完全想要的方式获取数据。

当使用 `Fetcher` 来服务数据时，您可以预期以下端点结构：

```python
@router.command(model="Example")
async def model_example(    # 创建一个异步端点
    cc: CommandContext,
    provider_choices: ProviderChoices,
    standard_params: StandardParams,
    extra_params: ExtraParams,
) -> OBBject:
    """示例数据."""
    return await OBBject.from_query(Query(**locals()))
```

让我们分解一下：

- `@router.command(...)` - 这告诉 OpenBB 平台这是一个命令。
- `model="Example"` - 这是您在 `<your_package_name>/<your_module_name>` 文件夹的 `__init__.py` 文件中定义的 `Fetcher` 字典键的名称。
- `cc: CommandContext` - 这包含一组用户和系统设置，在执行命令时非常有用 - 例如 api 密钥。
- `provider_choices: ProviderChoices` - 实现 `Example` `Fetcher` 的所有提供商。
- `standard_params: StandardParams` - 对所有实现 `Example` `Fetcher` 的提供商来说都是通用的标准参数。
- `extra_params: ExtraParams` - 它包含特定于提供商的参数，这些参数没有标准化。

您只需要将 `model` 参数更改为您定义的 `Fetcher` 字典键的名称，其他一切都将由 OpenBB 平台处理。

### 架构考虑

#### 重要类

#### 导入语句

```python

# `Data` 类
from openbb_core.provider.abstract.data import Data

# `QueryParams` 类
from openbb_core.provider.abstract.query_params import QueryParams

# `Fetcher` 类
from openbb_core.provider.abstract.fetcher import Fetcher

# `OBBject` 类
from openbb_core.app.model.obbject import OBBject

# `Router` 类
from openbb_core.app.router import Router

```

#### TET 模式

TET 模式是我们用来构建 `Fetcher` 类的模式。它代表 **转换，提取，转换**。
由于 OpenBB 平台有自己的标准化框架，并且数据提取器是非常重要的一部分，我们需要确保数据以一致的方式进行转换和提取，为了帮助我们做到这一点，我们想出了 **TET** 模式，它帮助我们更快地构建和发布，因为我们在如何构建 `Fetcher` 类上有了清晰的结构。

1. 转换 - `transform_query(params: Dict[str, Any])`：转换查询参数。给定一个 `params` 字典，这个方法应该返回转换后的查询参数作为 [`QueryParams`](openbb_platform/platform/provider/openbb_core/provider/abstract/query_params.py) 的子类，以便我们可以利用 pydantic 模型模式和验证进入下一步。这也可能是执行任何给定参数的转换的地方，例如，如果您想将空日期转换为 `datetime.now().date()`。
2. 提取 - `extract_data(query: ExampleQueryParams,credentials: Optional[Dict[str, str]],**kwargs: Any,) -> Dict`：向 API 端点发出请求并返回原始数据。给定转换后的查询参数、凭据和任何其他额外参数，这个方法应该以字典形式返回原始数据。
3. 转换 - `transform_data(query: ExampleQueryParams, data: Dict, **kwargs: Any) -> List[ExampleHistoricalData]`：将原始数据转换为定义的数据模型。给定转换后的查询参数（可能对某些过滤有用）、原始数据和任何其他额外参数，这个方法应该以 [`Data`](openbb_platform/platform/provider/openbb_core/provider/abstract/data.py) 的子类列表形式返回转换后的数据。

#### 错误

为确保一致的错误处理行为，我们的 API 依赖于以下约定。

| 状态码 | 异常 | 详情 | 描述 |
| -------- | ------- | ------- | ------- |
| 400 | `OpenBBError` 或 `OpenBBError` 的子类 | 自定义消息。 | 使用此显式引发自定义异常，如 `EmptyDataError`。 |
| 422 | `ValidationError` | `Pydantic` 错误字典消息。 | 自动引发以通知用户有关查询验证错误。查询之外的 ValidationError 默认使用状态码 500 处理。 |
| 500 | 未在上面覆盖的任何异常，例如 `ValueError`、`ZeroDivisionError` | 意外错误。 | 意外异常，最有可能是一个 bug。 |

#### 数据处理命令

数据处理命令是用于处理可能来自或不一定来自 OpenBB 平台的数据的命令。
为了创建一个足够通用以供任何扩展使用的数据处理框架，我们创建了一个名为 [`Data`](/openbb_platform/platform/provider/openbb_core/provider/abstract/data.py) 的特殊抽象类，所有标准化的（因此其子类）都将继承自它。

为什么这么重要？
这样我们就可以确保所有 `OBBject.results` 将在我们应用现成的数据处理命令时，如 `ta`、`qa` 或 `econometrics` 菜单，有一个共同的基础。

但 `Data` 类到底是什么？
它是一个从 `BaseModel` 继承的 pydantic 模型，并且可以包含任何数量的额外字段。实际上，它看起来像这样：

```python

>>> res = obb.equity.price.historical("AAPL")
>>> res.results[0]

AVEquityHistoricalData(date=2023-11-03 00:00:00, open=174.24, high=176.82, low=173.35, close=176.65, volume=79829246.0, vwap=None, adj_close=None, dividend_amount=None, split_coefficient=None)

```

> `AVEquityHistoricalData` 类是 `Data` 类的子类。

注意我们是如何通过索引来获取 `results` 列表的第一个元素的（如果我们想将其视为表格输出，则代表单个行）。这只是意味着我们从 `obb.equity.price.historical` 命令中获取 `AVEquityHistoricalData` 的 `List`。或者，我们也可以说这等同于 `List[Data]`！

这非常强大，因为现在我们可以将任何数据处理命令应用于 `results` 列表，而不必担心底层数据结构。
这就是为什么，在数据处理命令（如 `ta` 菜单）上，我们会在其函数签名中找到以下内容：

```python

def ema(
        self,
        data: Union[List[Data], pandas.DataFrame],
        target: str = "close",
        index: str = "date",
        length: int = 50,
        offset: int = 0,
        chart: bool = False,
    ) -> OBBject[List[Data]]:

    ...

```

> 注意 `data` 实际上可以是不同的类型，但我们现在将专注于 `List[Data]` 的情况。

这是否意味着我只能在使用 `Data` 的子类实例化的情况下使用数据处理命令？
一点也不！考虑以下示例：

```python

>>> from openbb_core.provider.abstract.data import Data
>>> my_data_item_1 = {"open": 1, "high": 2, "low": 3, "close": 4, "volume": 5, "date": "2020-01-01"}
>>> my_data_item_1_as_data = Data.model_validate(my_data_item_1)
>>> my_data_item_1_as_data

Data(open=1, high=2, low=3, close=4, volume=5, date=2020-01-01)

```

这意味着 `Data` 类足够智能，可以理解您正在传递一个字典，并且它会尝试为您验证它。
换句话说，如果您使用的是来自 OpenBB 平台的数据，您只需要确保它可以通过 `Data` 类解析，您就可以使用数据处理命令。
换句话说，想象一下您有一个 dataframe，您想用它与 `ta` 菜单一起使用。您可以这样做：

```python

>>> res = obb.equity.price.historical("AAPL")
>>> my_df = res.to_dataframe() # 是的，您可以将 OBBject.results 转换为 dataframe！
>>> my_records = df.to_dict(orient="records")

>>> obb.ta.ema(data=my_record)

OBBject

results: [{'close': 77.62, 'close_EMA_50': None}, {'close': 80.25, 'close_EMA_50': ... # 这又是 `List[Data]`

```

> 注意，为了这个示例，我们使用了 `OBBject.to_dataframe()` 方法来有一个示例 dataframe，但它可以是您拥有的任何其他 dataframe。

##### Python 接口

当在 Python 接口上使用 OpenBB 平台时，文档字符串和类型提示是您最好的朋友，因为它提供了关于如何使用命令的大量上下文。
看一个 `ta` 菜单上的例子：

```python

def ema(
        self,
        data: Union[List[Data], pandas.DataFrame],
        target: str = "close",
        index: str = "date",
        length: int = 50,
        offset: int = 0,
        chart: bool = False,
    ) -> OBBject[List[Data]]:

    ...

```

我们可以很容易地推断出 `ema` 命令接受 `List[Data]` 或 `pandas.DataFrame` 格式的数据。

> 注意，将来可能会添加其他类型。

##### API 接口

当在 API 接口上使用 OpenBB 平台时，类型比 Python 的要有限一些，例如，我们不能使用 `pandas.DataFrame` 作为类型。然而，相同的原则适用于 `Data` 的含义，即，任何给定的数据处理命令，它们被标记为 API 上的 POST 端点，将接受数据作为请求正文中的记录列表，即：

```json

[
    {
        "open": 80,
        "high": 80.69,
        "low": 77.37,
        "close": 77.62,
        "volume": 2487300
    }
    ...
]

```

## 贡献者指南

贡献者指南旨在作为 [开发者指南](#developer-guidelines) 的延续。它们不是替代品，而是扩展，专门针对那些寻求直接增强 OpenBB 平台代码库的人。贡献者熟悉这两组指南对于确保与 OpenBB 平台的和谐且富有成效的参与至关重要。

有很多方法可以为 OpenBB 平台做出贡献。您可以添加 [新的数据点](#getting_started-add-a-new-data-point)，添加 [新的命令](#openbb-platform-commands)，添加 [新的可视化](/openbb_platform/extensions/charting/README.md)，添加 [新的扩展](#getting_started-build-openbb-extensions)，修复错误，改进或创建文档等。

### 对贡献者的期望

1. 使用案例：
   - 确保您的贡献直接增强了 OpenBB 平台的功能或扩展生态系统。

2. 文档：
   - 所有代码贡献都应附带相关文档，包括贡献的目的、工作原理以及对现有功能所做的任何更改。
   - 如果您的贡献改变了 OpenBB 平台的行为，请更新任何现有文档。

3. 代码质量：
   - 您的代码应严格遵守 OpenBB 平台的编码标准和约定。
   - 确保您的代码清晰、可维护且组织得当。

4. 测试：
   - 所有贡献都必须经过彻底测试，以避免向 OpenBB 平台引入错误。
   - 贡献应包括相关的自动化测试（单元和集成），任何新功能都应附带其测试用例。

5. 性能：
   - 您的贡献应针对性能进行优化，不应降低 OpenBB 平台的整体效率。
   - 解决任何潜在的瓶颈，并确保可扩展性。

6. 协作：
   - 积极与 OpenBB 开发团队合作，确保您的贡献与平台的发展路线图和标准一致。
   - 欢迎反馈，并对社区的审查和建议持开放态度，根据需要进行修订。

### 质量保证

我们坚信质量保证 (QA) 流程，并希望确保添加到 OpenBB 平台的所有扩展都是高质量的。为确保这一点，我们有一套 QA 工具可供您使用，以测试您的扩展。

首先，我们拥有半自动化创建单元和集成测试的工具。

> QA 工具仍在开发中，我们正在不断改进它们。

#### 单元测试

每个 `Fetcher` 都配备了一个 `test` 方法，确保它正确实现并且返回预期的数据。它还确保所有类型都是正确的，并且数据是有效的。

要为您的 Fetchers 创建单元测试，您可以运行以下命令：

```bash
python openbb_platform/providers/tests/utils/unit_tests_generator.py
```

> 注意，您应该从存储库的根目录运行此文件。
> 请注意，必须存在 `tests` 文件夹才能生成测试。

自动单元测试生成将为给定提供商中所有可用的 fetchers 添加单元测试。

要录制单元测试，您可以运行以下命令：

```bash
pytest <path_to_the_unit_test_file> --record=all
```

> 注意，有时可能需要手动干预。例如，调整顶级导入之外的导入或为特定 fetcher 添加特定参数。

#### 集成测试

集成测试比单元测试更复杂，因为我们想要测试 Python 接口和 API 接口。为此，我们有两个脚本可以帮助您生成集成测试。

要为 Python 接口生成集成测试，您可以运行以下命令：

```bash
python openbb_platform/extensions/tests/utils/integration_tests_generator.py
```

要为 API 接口生成集成测试，您可以运行以下命令：

```bash
python openbb_platform/extensions/tests/utils/integration_tests_api_generator.py
```

当测试 API 接口时，您需要在运行测试之前在本地运行 OpenBB 平台。为此，您可以运行以下命令：

```bash
uvicorn openbb_platform.core.openbb_core.api.rest_api:app --host 0.0.0.0 --port 8000 --reload
```

这些自动化测试是减少您需要编写的代码量的好方法，但它们不是手动测试的替代品，可能需要调整。这就是为什么我们有单元测试来测试生成的集成测试，以确保它们涵盖所有提供商和参数。

要运行测试，我们可以执行以下操作：

- 仅单元测试：

```bash
pytest openbb_platform -m "not integration"
```

- 仅集成测试：

```bash
pytest openbb_platform -m integration
```

- 集成和单元测试：

```bash
pytest openbb_platform
```

#### 导入时间

我们的目标是使包的导入时间尽可能短。为了衡量这一点，我们使用 `tuna`。

- <https://pypi.org/project/tuna/>

要通过模块可视化导入时间分解并找到潜在的瓶颈，请从 `openbb_platform` 目录运行以下命令：

```bash
pip install tuna
python -X importtime openbb/__init__.py 2> import.log
tuna import.log
```

### 分享您的扩展

我们鼓励您与社区分享您的扩展。您可以通过将其发布到 PyPI 来实现。

#### 将您的扩展发布到 PyPI

要将您的扩展发布到 PyPI，您需要拥有 PyPI 帐户和 PyPI API 令牌。

##### 设置

创建帐户并从 <https://pypi.org/manage/account/token/> 获取 API 令牌。
使用以下命令存储令牌：

```bash
poetry config pypi-token.pypi pypi-YYYYYYYY
```

##### 版本上线

`cd` 进入包含您的扩展 `pyproject.toml` 的目录，并确保 `pyproject.toml` 指定了您要发布的版本标签，然后运行。

```bash
poetry build
```

这将在目录中创建一个 `/dist` 文件夹，其中包含与要发布的版本匹配的 `.whl` 和 `tar.gz` 文件。

如果您想本地测试您的包，可以这样做：

```bash
pip install dist/openbb_[FILE_NAME].whl
```

##### 发布

要将您的包发布到 PyPI，请运行：

```bash
poetry publish
```

现在，您可以使用以下命令从 PyPI pip 安装您的包：

```bash
pip install openbb-some_ext
```

### 管理扩展

要安装托管在 PyPI 上的扩展，请使用 `pip install <extension>` 命令。

要安装本地开发的扩展，请确保它包含 `pyproject.toml` 文件，然后使用 `pip install <extension>` 命令。

> 要使用 pip 安装可编辑模式的扩展，请添加 `-e` 参数。

或者，对于本地扩展，您可以在 `dev_install.py` 文件的 `LOCAL_DEPS` 变量中添加以下行：

```toml
# 如果这是社区依赖项，请在 "Community dependencies" 下添加此内容，
# 并使用额外的参数 optional = true
openbb-extension = { path = "<relative-path-to-the-extension>", develop = true }
```

现在，您可以使用 `python dev_install.py [-e]` 命令安装本地扩展。

#### 添加扩展作为依赖项

要将 `openbb-qa` 扩展添加为依赖项，您需要将其添加到 `pyproject.toml` 文件中：

```toml
[tool.poetry.dependencies]
openbb-qa = "^0.0.0a2"
```

然后，您可以按照上述相同的过程安装扩展。

### 编写代码并提交

#### 如何创建 PR？

要为 OpenBB 平台创建 PR，您需要 fork 存储库并创建一个新分支。

1. 创建您的功能分支，例如 `git checkout -b feature/AmazingFeature`
2. 使用 `git status` 检查您触及的文件。
3. 将您想要提交的文件暂存，例如：
   `git add openbb_platform/platform/core/openbb_core/app/constants.py`。
   注意：**不要**添加任何包含个人信息的文件。
4. 编写一个简洁的提交消息，少于 50 个字符，例如 `git commit -m "有意义的提交消息"`。如果您的 PR 解决了用户提出的问题，您可以通过在提交消息中添加 #ISSUE_NUMBER 来指定该问题，以便这些被链接起来。注意：如果您安装了预提交钩子，并且其中一个格式化程序重新格式化了您的代码，您将需要返回到第 3 步来添加这些。

##### 分支命名约定

接受的分支命名约定是：

- `feature/feature-name`
- `hotfix/hotfix-name`

这些分支只能指向 `develop` 分支的 PR。
