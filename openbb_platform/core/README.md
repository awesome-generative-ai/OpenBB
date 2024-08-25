# OpenBB平台 - 核心组件

## 概览

核心扩展是OpenBB平台的基础组件。它封装了必要的功能，并为其他扩展提供了基础设施基础。此扩展对于维护平台的完整性和标准化至关重要。

## 主要特性

- **标准化数据模型** (`Data` 类): 一个灵活且动态的Pydantic模型，能够处理各种数据结构。
- **标准化查询参数** (`QueryParams` 类): 一个用于处理不同提供商查询的Pydantic模型。
- **动态字段支持**: 能够处理未定义字段，为数据处理提供多样性。
- **强大的数据验证**: 利用Pydantic的验证特性确保数据完整性。
- **API路由机制** (`Router` 类): 简化定义API路由和端点的过程 - 即开即用的Python和Web端点。

## 开始使用

### 先决条件

- Python 3.8或更高版本。
- 对FastAPI和Pydantic的熟悉。

### 安装

通过pip安装：

```bash
pip install openbb-core
```

> 注意，openbb-core是OpenBB平台的基础设施组件。它不打算作为独立的包使用。

### 使用

核心扩展用作构建和集成新的数据源、提供商和扩展到OpenBB平台的基础。它提供了标准化和处理数据所需的类和结构。

### 贡献

我们欢迎贡献！如果你想贡献，请：

- 遵循现有的编码标准和约定。
- 编写清晰、有文档的代码。
- 确保你的代码不会对性能产生负面影响。
- 彻底测试你的贡献。

请参阅我们的[贡献指南](https://docs.openbb.co/platform/developer_guide/contributing)。

### 协作

与开发团队和社区进行互动。对反馈和协作讨论持开放态度。

### 支持

如需支持、问题或更多信息，请访问[OpenBB平台文档](https://docs.openbb.co/platform)。

### 许可证

本项目根据MIT许可证获得许可 - 请参阅[LICENSE.md](https://github.com/awesome-generative-ai/OpenBB/blob/main/LICENSE) 文件了解详细信息。
