# Compound Engineering Flutter

Flutter/Dart AI 开发工具集 -- Claude Code 插件。

> **复合工程理念**：每一次工程工作都应该让后续工作变得更简单。

## 组件概览

| 类别        | 数量 | 说明                   |
| ----------- | ---- | ---------------------- |
| Agents      | 9    | 专业 AI 审查和设计代理 |
| Commands    | 6    | 工作流命令             |
| Skills      | 5    | 领域知识技能           |
| MCP Servers | 1    | Context7 框架文档查询  |

## Agents

### 代码审查

| Agent                       | 说明                                                 |
| --------------------------- | ---------------------------------------------------- |
| `flutter-code-reviewer`     | 全面审查 Flutter/Dart 代码的规范性、清晰度和可维护性 |
| `dart-style-reviewer`       | 以 Effective Dart 为标准审查代码风格                 |
| `flutter-security-reviewer` | 审查应用安全性：数据存储、网络通信、密钥管理         |

### 架构设计

| Agent                         | 说明                                             |
| ----------------------------- | ------------------------------------------------ |
| `widget-architect`            | Widget 架构设计，组件拆分和复用体系              |
| `state-management-advisor`    | 状态管理选型和架构设计（Bloc/Riverpod/Provider） |
| `flutter-navigation-designer` | 路由架构设计，GoRouter 和深链接                  |

### 质量保障

| Agent                           | 说明                                          |
| ------------------------------- | --------------------------------------------- |
| `flutter-performance-optimizer` | 渲染性能、内存管理、启动速度和包体积优化      |
| `flutter-test-strategist`       | 测试策略设计，Widget 测试、单元测试和集成测试 |

## Commands

### 工作流命令

| 命令              | 说明                                       |
| ----------------- | ------------------------------------------ |
| `/flutter:review` | 多维度代码审查（风格+架构+状态+性能+安全） |
| `/flutter:plan`   | 功能需求分析和实现计划制定                 |
| `/flutter:work`   | 按计划逐步执行开发任务                     |

### 效率命令

| 命令                   | 说明                                                  |
| ---------------------- | ----------------------------------------------------- |
| `/flutter-lfg`         | 全自动工程工作流：计划 -> 开发 -> 审查                |
| `/flutter-new-feature` | 快速生成功能模块脚手架（支持 Bloc/Riverpod/Provider） |
| `/flutter-analyze`     | 项目健康检查：静态分析+依赖+覆盖率                    |

## Skills

| Skill                        | 说明                                            |
| ---------------------------- | ----------------------------------------------- |
| `flutter-clean-architecture` | Clean Architecture 分层架构实践指南             |
| `dart-style`                 | Dart 编码规范速查（基于 Effective Dart）        |
| `widget-patterns`            | Widget 设计模式：布局、列表、响应式、表单、动画 |
| `flutter-testing`            | 测试最佳实践：单元/Widget/集成/Golden/Mock      |
| `flutter-app-setup`          | 项目初始化和团队配置：Lint、CI/CD、主题、环境   |

## MCP Servers

| 服务       | 类型 | 说明                                       |
| ---------- | ---- | ------------------------------------------ |
| `context7` | HTTP | 框架文档实时查询（包括 Flutter/Dart 文档） |

## 安装

```bash
# 通过 Claude Code 安装
claude /plugin marketplace add <marketplace-url>
claude /plugin install compound-engineering-flutter
```

## 使用示例

```bash
# 全自动开发工作流
claude /flutter-lfg "实现商品列表页面，使用 Bloc 状态管理"

# 代码审查
claude /flutter:review latest

# 生成功能脚手架
claude /flutter-new-feature product --bloc

# 项目健康检查
claude /flutter-analyze --fix

# 使用特定 Agent
claude agent flutter-code-reviewer "审查购物车模块的代码"
claude agent widget-architect "设计电商首页的 Widget 架构"
claude agent state-management-advisor "分析这个项目应该用什么状态管理方案"
```

## 版本历史

查看 [CHANGELOG.md](CHANGELOG.md) 了解完整的版本历史。

## 许可

MIT
