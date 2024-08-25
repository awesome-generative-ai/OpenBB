# OpenBB图表扩展

该扩展为OpenBB平台提供了一个图表库。

该库包括：

- 基于Plotly的图表基础设施
- 一组图表组件
- 为内置OpenBB扩展构建的一组预构建图表

>[!NOTE]
> 图表库是一个`OBBject`扩展，这意味着您将在每个命令结果上拥有它公开的功能。

## 安装

要安装该扩展，请在此文件夹中运行以下命令：

```bash
pip install openbb-charting
```

## Linux上的PyWry依赖性

PyWry依赖性处理在单独窗口中显示交互式图表和表格。它与OpenBB图表扩展一起自动安装。

在使用Linux发行版时，PyWry依赖性需要首先安装某些依赖项。

- 基于Debian/Ubuntu/Mint的系统：
`sudo apt install libwebkit2gtk-4.0-dev`

- Arch Linux/Manjaro：
`sudo pacman -S webkit2gtk`

- Fedora：
`sudo dnf install gtk3-devel webkit2gtk3-devel`

## 使用方法

要使用该扩展，请运行任何OpenBB平台端点，并设置`chart`参数为`True`。

以下是在Python接口中可能看起来的样子：

```python
from openbb import obb
equity_data = obb.equity.price.historical(symbol="TSLA", chart=True)
```

这将生成一个包含`chart`属性的`OBBject`对象，其中包含Plotly JSON数据。

为了显示图表，您需要调用`show()`方法：

```python
equity_data.show()
```

> 注意：`show()`方法当前只能在Jupyter Notebook中或在正确初始化了基于PyWry的后端的独立Python脚本中工作。

或者，您可以利用`openbb-charting`是一个`OBBject`扩展，并使用其可用的方法。

```python
from openbb import obb
res = obb.equity.price.historical("AAPL")
res.charting.show()
```

上述代码将产生与前一个示例相同的效果。

### 发现可用的图表

并非所有端点当前都支持图表扩展。要发现哪些端点受支持，您可以运行以下命令：

```python
from openbb_charting import Charting
Charting.functions()
```

### 使用`to_chart`方法

`to_chart`函数应被视为高级功能，因为它要求用户对图表扩展和`OpenBBFigure`类有良好的理解。

用户可以使用任何数量的`**kwargs`，这些将传递给`PlotlyTA`类，以便使用自定义指标构建自定义可视化。

> 注意，此方法在一定程度上只能与非标准化数据一起有限工作。
> 另外，它目前仅设计为处理时间序列(OHLCV)数据。

示例用法：

- 绘制带有TA指标的时间序列

  ```python
    from openbb import obb
    res = obb.equity.price.historical("AAPL")

    indicators = dict(
        sma=dict(length=[20,30,50]),
        adx=dict(length=14),
        rsi=dict(length=14),
        macd=dict(fast=12, slow=26, signal=9),
        bbands=dict(length=20, std=2),
        stoch=dict(length=14),
        ema=dict(length=[20,30,50]),
    )
    res.charting.to_chart(**{"indicators": indicators})
  ```

- 获取所有可用的指标

    ```python
    # 如果您已经有了命令结果
    res.charting.indicators

    # 或者如果您想以独立的方式知道
    from openbb_charting import Charting
    Charting.indicators()
    ```

## 向现有平台命令添加可视化

要向现有命令添加可视化，您需要在`pyproject.toml`文件中添加一个`poetry`插件。语法应该是以下内容：

```toml
[tool.poetry.plugins."openbb_charting_extension"]
my_extension = "openbb_my_extension.my_extension_views:MyExtensionViews"
```

其中`openbb_charting_extension`是**强制性的**，否则图表扩展将无法找到可视化。

建议的`my_extension_views`模块结构如下：

```python
"""Views for MyExtension."""

from typing import Any, Dict, Tuple

from openbb_charting.charts.price_historical import price_historical
from openbb_charting.core.openbb_figure import OpenBBFigure


class MyExtensionViews:
    """MyExtension Views."""

    @staticmethod
    def my_extension_price_historical(
        **kwargs,
    ) -> Tuple[OpenBBFigure, Dict[str, Any]]:
        """MyExtension Price Historical Chart."""
        return price_historical(**kwargs)
```

> 注意`my_extension_views`位于`openbb_my_extension`包下。

之后，您需要将可视化添加到新的`MyExtensionViews`类中。匹配端点与相应图表函数的约定如下：

- `/equity/price/historical` -> `equity_price_historical`
- `/technical/ema` -> `technical_ema`
- `/my_extension/price_historical` -> `my_extension_price_historical`

当您在图表路由器文件中看到图表函数时，可以将可视化添加到其中。

实现应该利用已经存在的类和方法来实现，即：

- `OpenBBFigure`
- `PlotlyTA`

请注意，每个图表函数的返回应该尊重已经定义的返回类型：`Tuple[OpenBBFigure, Dict[str, Any]]`。

返回的元组包含一个`OpenBBFigure`，这是一个交互式的plotly图形，可以用于Python解释器，以及一个`Dict[str, Any]`，其中包含API利用的原始数据。

完成图表函数的实现后，您可以使用Python接口或API获取图表。为此，您只需要将已经可用的`chart`参数设置为`True`。
或访问`OBBject`对象的`charting`属性：`my_obbject.charting.show()`。
