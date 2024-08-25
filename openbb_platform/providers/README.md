# 数据提供商

在这个文件夹中，您可以找到由OpenBB创建或支持的数据提供商。

## 推荐结构

每个数据提供商都位于一个目录中，具有以下结构：

```{.bash}
openbb_platform
└───providers
    └───<provider_name>
        |   README.md
        │   pyproject.toml
        │   poetry.lock
        |───tests
        └───openbb_<provider_name>
            │   __init__.py
            |───models
            |   |───<some model>.py
            |   └───...
            └───utils
                |───<some helper>.py
                └───...
```

这些模型定义了用于查询提供商端点和存储响应数据的数据结构。

更多详细信息，请参阅[贡献文件](../CONTRIBUTING.md)