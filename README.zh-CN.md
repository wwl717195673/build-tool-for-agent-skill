# 构建 AI Agent 工具

一份全面的指南，帮助您遵循 Anthropic 工程最佳实践和模型上下文协议 (MCP) 规范标准来构建高效的 AI Agent 工具。

## 概述

本项目提供了创建 AI Agent 可有效使用的工具的综合指南。涵盖从设计原则到实施最佳实践的完整工具开发生命周期。

## 核心原则

### 1. 选择合适的工具
设计能够完成特定任务的工具，而不是暴露原始数据。工具应该帮助 Agent 完成有意义的工作，而不仅仅是检索信息。

### 2. 对工具进行命名空间管理
当 Agent 可以访问多个 MCP 服务器时，使用服务前缀（例如 `asana_search`、`jira_search`）来组织工具。

### 3. 返回有意义的上下文
优先考虑上下文相关性而非灵活性。使用语义字段如 `name`、`image_url` 和 `file_type`，而不是低级标识符如 `uuid`。

### 4. 优化 Token 效率
- 实现分页
- 使用范围选择
- 应用过滤
- 设置合理的默认限制

### 5. 优化工具描述
撰写描述时就像描述新员工的工作一样。清楚解释隐含的上下文、专业格式和术语。

## 开发工作流

```
构建原型 → 运行评估 → 分析结果 → 改进工具 → 重复
```

1. **构建原型**：创建工具原型并在本地测试。打包到本地 MCP 服务器中。
2. **运行评估**：生成需要多次工具调用的评估任务。收集指标。
3. **分析改进**：审查评估结果并优化工具。
4. **重复**：继续迭代以提高有效性。

## MCP 工具定义

### 必填字段

| 字段 | 描述 |
|------|------|
| `name` | 工具的唯一标识符 |
| `description` | 工具功能的可读描述 |
| `inputSchema` | 定义预期参数的 JSON Schema |

### 示例

```json
{
  "name": "get_weather",
  "description": "获取指定位置的当前天气信息。",
  "inputSchema": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "城市名称或邮政编码（例如 'New York' 或 '10001'）"
      }
    },
    "required": ["location"]
  }
}
```

## 最佳实践

### 输入 Schema
- 为所有参数包含全面的描述（至少 50 个字符）
- 指定精确的类型（string、number、enum、array）
- 定义有效值和范围

### 错误处理
- 返回描述性错误消息，帮助 Agent 理解并恢复
- 对协议错误使用标准 JSON-RPC 错误代码

### 安全性
- 验证所有工具输入
- 实现访问控制
- 对工具调用进行速率限制
- 清理输出

## 评估

### 强任务示例
- "安排下周与 Jane 的会议，讨论项目..."
- "查找客户 ID 9182 的所有相关日志条目..."

### 收集的指标
- 顶级准确率
- 总运行时间
- 工具调用次数
- Token 消耗
- 工具错误

## 参考资源

- [模型上下文协议文档](https://modelcontextprotocol.io/docs/getting-started/intro)
- [MCP SDK](https://modelcontextprotocol.io/docs/sdk)
- [Anthropic API 文档](https://docs.anthropic.com/llms.txt)
- [工具评估 Cookbook](https://github.com/anthropics/anthropic-cookbook/blob/main/tool_evaluation/tool_evaluation.ipynb)

## 许可证

本指南仅供教育目的。
