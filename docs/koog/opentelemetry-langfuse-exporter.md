# Langfuse 导出器

Koog 内置支持将代理轨迹导出到 [Langfuse](https://langfuse.com/)，这是一个用于 AI 应用程序可观测性和分析的平台。
通过 Langfuse 集成，您可以可视化、分析和调试 Koog 代理如何与大型语言模型（LLM）、API 和其他组件进行交互。

关于 Koog 的 OpenTelemetry 支持的背景信息，请参见 [OpenTelemetry 支持](https://docs.koog.ai/opentelemetry-support/)。

---

### 设置说明

1. 创建一个 Langfuse 项目。请按照位于 [在 Langfuse 中创建新项目](https://langfuse.com/docs/get-started#create-new-project-in-langfuse) 的设置指南操作。
2. 获取 API 凭据。按照 [Langfuse API 密钥在哪里？](https://langfuse.com/faq/all/where-are-langfuse-api-keys) 中所述，获取您的 Langfuse `public key` 和 `secret key`。
3. 将 Langfuse 主机、`public key` 和 `secret key` 传递给 Langfuse 导出器。
这可以通过将它们作为参数传递给 `addLangfuseExporter()` 函数来完成，
或者通过设置环境变量，如下所示：

```bash
   export LANGFUSE_HOST="https://cloud.langfuse.com"
   export LANGFUSE_PUBLIC_KEY="<your-public-key>"
   export LANGFUSE_SECRET_KEY="<your-secret-key>"
```

## 配置

要启用 Langfuse 导出，请安装 **OpenTelemetry 特性**并添加 `LangfuseExporter`。
该导出器底层使用 `OtlpHttpSpanExporter` 将轨迹发送到 Langfuse 的 OpenTelemetry 端点。

### 示例：带 Langfuse 追踪的代理

<!--- INCLUDE
import ai.koog.agents.core.agent.AIAgent
import ai.koog.agents.features.opentelemetry.feature.OpenTelemetry
import ai.koog.agents.features.opentelemetry.integration.langfuse.addLangfuseExporter
import ai.koog.prompt.executor.clients.openai.OpenAIModels
import ai.koog.prompt.executor.llms.all.simpleOpenAIExecutor
import kotlinx.coroutines.runBlocking
-->
```kotlin
fun main() = runBlocking {
    val apiKey = "api-key"
    
    val agent = AIAgent(
        executor = simpleOpenAIExecutor(apiKey),
        llmModel = OpenAIModels.CostOptimized.GPT4oMini,
        systemPrompt = "You are a code assistant. Provide concise code examples."
    ) {
        install(OpenTelemetry) {
            addLangfuseExporter()
        }
    }

    println("Running agent with Langfuse tracing")

    val result = agent.run("Tell me a joke about programming")

    println("Result: $result
See traces on the Langfuse instance")
}
```
<!--- KNIT example-langfuse-exporter-01.kt -->

## 追踪了什么

启用后，Langfuse 导出器会捕获与 Koog 通用 OpenTelemetry 集成相同的 span，包括：

- **代理生命周期事件**：代理启动、停止、错误
- **大型语言模型（LLM）交互**：提示、响应、token 使用、延迟
- **工具调用**：工具调用的执行轨迹
- **系统上下文**：元数据，例如模型名称、环境、Koog 版本

Koog 还会捕获 Langfuse 所需的 span 属性，以显示 [代理图](https://langfuse.com/docs/observability/features/agent-graphs)。

在 Langfuse 中可视化时，轨迹显示如下：
![Langfuse traces](img/opentelemetry-langfuse-exporter-light.png#only-light)
![Langfuse traces](img/opentelemetry-langfuse-exporter-dark.png#only-dark)

关于 Langfuse OpenTelemetry 追踪的更多详情，请参见：
[Langfuse OpenTelemetry 文档](https://langfuse.com/integrations/native/opentelemetry#opentelemetry-endpoint)。

---

## 故障排除

### Langfuse 中未出现轨迹
- 仔细检查 `LANGFUSE_HOST`、`LANGFUSE_PUBLIC_KEY` 和 `LANGFUSE_SECRET_KEY` 是否已在您的环境中设置。
- 如果在自托管的 Langfuse 上运行，确认 `LANGFUSE_HOST` 可从您的应用程序环境中访问。
- 验证 `public/secret key` 对是否属于正确的项目。