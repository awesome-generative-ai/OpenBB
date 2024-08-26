# OpenBB 工作流程

这个目录包含了 OpenBB 🦋 项目的流程。工作流程包括：

## 📑 部署到 GitHub Pages

这个 GitHub Actions 工作流程负责构建文档并将其部署到 GitHub Pages。当仓库的 `main` 或 `release` 分支有新变更被推送时，此工作流程会被触发，并将文档发布到 GitHub Pages。

## 分支名称检查

目标：在合并前检查拉取请求分支名称是否遵循 GitFlow 命名规范。

触发条件：目标分支为 develop 或 main 的拉取请求事件。

检查的分支：拉取请求的源分支和目标分支。

步骤：
1. 提取分支名称：使用 jq 工具，从拉取请求事件中提取源分支和目标分支名称。然后将分支名称存储在环境变量中，并打印为输出。
2. 显示源分支和目标分支的输出结果：将源分支和目标分支名称打印到控制台。
3. 检查 develop PR 的分支名称：如果目标分支是 develop，则检查源分支是否符合 GitFlow 命名规范的正则表达式。如果分支名称无效，将消息打印到控制台，并以状态码 1 退出工作流程。
4. 检查 main PR 的分支名称：如果目标分支是 main，则检查源分支是否为热修复或发布分支。如果分支名称无效，将消息打印到控制台，并以状态码 1 退出工作流程。

注意：GitFlow 分支命名规范如下：

- 功能分支：feature/<feature-name>
- 热修复分支：hotfix/<hotfix-name>
- 发布分支：release/<major.minor.patch>(rc<number>)

## 部署到 PyPI - 夜间版

此工作流程用于将 OpenBB 平台 CLI 的最新版本发布到 PyPI。工作流程由 GitHub Action 计划事件在 UTC+0 每天触发。

它首先通过将 `pyproject.toml` 文件更新为预设的版本字符串 `<currentVersion>.dev<date>`，其中 `<date>` 表示当前日期的 8 位数字。

然后，代码安装 `pypa/build` 并使用 `python -m build` 在 `dist/` 目录中创建二进制轮和源代码 tarball。

最后，它使用 PyPA 特定的动作 `gh-action-pypi-publish` 将创建的文件发布到 PyPI。

## 部署 OpenBB 平台到测试 PyPI

GitHub Action 代码 `Deploy to PyPI` 用于将 Python 项目部署到 PyPI（Python Package Index）和 TestPyPI，TestPyPI 是一个用于测试的单独的包索引。代码在两个事件上触发：

1. 推送事件：每当有推送到 `release/*` 和 `main` 分支时，代码就会被触发。
2. 工作流程调度事件：代码可以通过工作流程调度事件手动触发。

代码将并发设置为 `group`，并将选项 `cancel-in-progress` 设置为 `true`，以确保在触发另一个作业时取消同一 `group` 中正在运行的作业。

代码包含两个作业，`deploy-test-pypi` 和 `deploy-pypi`，这两个作业具有相同但略有变化的步骤。

`deploy-test-pypi` 作业仅在推送的分支以 `refs/heads/release/` 开头时触发。此作业设置 Python 环境，使用 `pip` 安装 `build` 包，使用 `build` 构建二进制轮和源代码 tarball，并最终使用 `pypa/gh-action-pypi-publish@release/v1` GitHub Action 将分发到 TestPyPI。访问 TestPyPI 的 `password` 存储为名为 `TEST_PYPI_API_TOKEN` 的秘密。

类似地，`deploy-pypi` 作业仅在推送的分支以 `refs/heads/main` 开头时触发。此作业遵循与 `deploy-test-pypi` 相同的步骤，但将分发到 PyPI 而不是 TestPyPI。访问 PyPI 的 `password` 存储为名为 `PYPI_API_TOKEN` 的秘密。

注意：代码使用 `pypa/build` 包构建二进制轮和源代码 tarball，并使用 `pypa/gh-action-pypi-publish@release/v1` GitHub Action 将分发到 PyPI 和 TestPyPI。

## 起草发布

这个 GitHub Actions 工作流程旨在自动生成和更新 GitHub 仓库中的草稿发布。工作流程在手动调度时触发，允许你控制草稿发布的更新时间。

## 🧹 通用 Linting

这个 GitHub Actions 工作流程负责在代码库上运行 linting 检查。此工作流程在 `opened`、`synchronize` 和 `edited` 等拉取请求事件上触发，以及在以 `feature/`、`hotfix/` 或 `release/` 开头的分支上的推送事件。工作流程还设置了多个环境变量，并使用 Github Actions 缓存来提高性能。

它由两个作业组成：`code-linting` 和 `markdown-link-check`。

第一个作业 `code-linting` 在 Ubuntu 机器上运行，并对仓库中的代码执行多个 linting 任务，包括：

- 从仓库检出代码
- 设置 Python 3.9
- 安装多个必要的 Python 包以进行 linting 任务
- 运行 `bandit` 检查安全漏洞
- 运行 `black` 检查代码格式化
- 运行 `codespell` 检查注释、字符串和小变量名称的拼写
- 运行 `ruff` 检查 Python 的使用
- 运行 `pylint` 对代码进行静态分析
- 运行 `mypy` 检查类型注释
- 运行 `pydocstyle` 检查文档字符串

第二个作业 `markdown-link-check` 在 Ubuntu 机器上运行，并对仓库中的 markdown 文件进行 linting。它使用 Docker 容器 `avtodev/markdown-lint` 执行 linting。

## 🏷️ 拉取请求标签

自动标记拉取请求。

## 🚉 集成测试平台（API）

运行 `openbb_platform` API 集成测试，

## 🖥️ CLI 单元测试

运行 `cli` 目录单元测试。

## 🚉 平台单元测试

运行 `openbb_platform` 目录单元测试 - 提供商、扩展等。
