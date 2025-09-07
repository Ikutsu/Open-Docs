# W&B Weave 导出器

Koog 内置支持将 agent 追踪导出到 [W&B Weave](https://wandb.ai/site/weave/)，这是一个来自 Weights & Biases 的开发者工具，用于 AI 应用程序的可观测性和分析。通过 Weave 集成，您可以捕获 prompt、completion、系统上下文和执行追踪，并直接在您的 W&B workspace 中可视化它们。

关于 Koog 的 OpenTelemetry 支持的背景信息，请参见 [OpenTelemetry support](https://docs.koog.ai/opentelemetry-support/)。

---

## 设置说明

1.  在 [https://wandb.ai](https://wandb.ai) 注册一个 W&B 账户
2.  从 [https://wandb.ai/authorize](https://wandb.ai/authorize) 获取您的 API key。
3.  通过访问 [https://wandb.ai/home](https://wandb.ai/home) 上的 W&B 控制面板，找到您的 entity 名称。您的 entity 通常是个人账户的用户名，或者是团队/组织的名称。
4.  为您的 project 定义一个名称。您不必预先创建 project，当第一个追踪发送时它将自动创建。
5.  将 Weave entity、project 名称和 API key 传递给 Weave 导出器。这可以通过将它们作为参数提供给 `addWeaveExporter()` 函数来完成，也可以通过设置环境变量来完成，如下所示：

```bash
export WEAVE_API_KEY="<your-api-key>"
export WEAVE_ENTITY="<your-entity>"
export WEAVE_PROJECT_NAME="koog-tracing"
```

## 配置

要启用 Weave 导出，请安装 **OpenTelemetry 特性**并添加 `WeaveExporter`。该导出器通过 `OtlpHttpSpanExporter` 使用 Weave 的 OpenTelemetry 端点。

### 示例：带 Weave 追踪的 agent

<!--- INCLUDE
import ai.koog.agents.core.agent.AIAgent
import ai.koog.agents.features.opentelemetry.feature.OpenTelemetry
import ai.koog.agents.features.opentelemetry.integration.weave.addWeaveExporter
import ai.koog.prompt.executor.clients.openai.OpenAIModels
import ai.koog.prompt.executor.llms.all.simpleOpenAIExecutor
import kotlinx.coroutines.runBlocking
-->
```kotlin
fun main() = runBlocking {
    val apiKey = "api-key"
    val entity = System.getenv()["WEAVE_ENTITY"] ?: throw IllegalArgumentException("WEAVE_ENTITY is not set")
    val projectName = System.getenv()["WEAVE_PROJECT_NAME"] ?: "koog-tracing"
    
    val agent = AIAgent(
        executor = simpleOpenAIExecutor(apiKey),
        llmModel = OpenAIModels.CostOptimized.GPT4oMini,
        systemPrompt = "You are a code assistant. Provide concise code examples."
    ) {
        install(OpenTelemetry) {
            addWeaveExporter()
        }
    }

    println("Running agent with Weave tracing")

    val result = agent.run("Tell me a joke about programming")

    println("Result: $result
See traces on https://wandb.ai/$entity/$projectName/weave/traces")
}
```
<!--- KNIT example-weave-exporter-01.kt -->

## 追踪了什么

启用后，Weave 导出器捕获的 span 与 Koog 的通用 OpenTelemetry 集成捕获的相同，包括：

-   **Agent 生命周期事件**：agent 启动、停止、错误
-   **LLM 交互**：prompt、completion、延迟
-   **工具调用**：工具调用的执行追踪
-   **系统上下文**：元数据，例如模型名称、环境、Koog 版本

在 W&B Weave 中可视化时，追踪显示如下：
![W&B Weave traces](img/opentelemetry-weave-exporter-light.png#only-light)
![W&B Weave traces](img/opentelemetry-weave-exporter-dark.png#only-dark)

关于更多详情，请参见官方 [Weave OpenTelemetry Docs](https://weave-docs.wandb.ai/guides/tracking/otel/)。

---

## 故障排除

### Weave 中未显示追踪
-   确认 `WEAVE_API_KEY`、`WEAVE_ENTITY` 和 `WEAVE_PROJECT_NAME` 已在您的环境中设置。
-   确保您的 W&B 账户具有访问指定 entity 和 project 的权限。

### 认证错误
-   检测您的 `WEAVE_API_KEY` 是否有效。
-   API key 必须具有为所选 entity 写入追踪的权限。

### 连接问题
-   确保您的环境具有访问 W&B 的 OpenTelemetry 摄入端点的网络权限。