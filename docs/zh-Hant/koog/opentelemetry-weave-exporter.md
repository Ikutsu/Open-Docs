# W&B Weave 匯出器

Koog 提供內建支援，可將代理程式追蹤匯出至 [W&B Weave](https://wandb.ai/site/weave/)。W&B Weave 是 Weights & Biases 推出的一款開發者工具，用於 AI 應用程式的可觀察性與分析。透過 Weave 整合，您可以擷取提示、完成內容、系統上下文和執行追蹤，並直接在您的 W&B 工作區中將其視覺化。

有關 Koog 對 OpenTelemetry 支援的背景資訊，請參閱 [OpenTelemetry 支援](https://docs.koog.ai/opentelemetry-support/)。

---

## 設定說明

1.  在 [https://wandb.ai](https://wandb.ai) 建立 W&B 帳戶。
2.  從 [https://wandb.ai/authorize](https://wandb.ai/authorize) 取得您的 API key。
3.  透過造訪 [https://wandb.ai/home](https://wandb.ai/home) 上的 W&B 儀表板來找到您的實體名稱。如果這是個人帳戶，您的實體通常是您的使用者名稱；如果是團隊帳戶，則是您的團隊/組織名稱。
4.  為您的專案定義一個名稱。您不需要事先建立專案，當第一個追蹤被發送時，它將會自動建立。
5.  將 Weave 實體、專案名稱和 API key 傳遞給 Weave 匯出器。這可以透過將它們作為參數提供給 `addWeaveExporter()` 函數來完成，或者透過設定環境變數，如下所示：

```bash
export WEAVE_API_KEY="<your-api-key>"
export WEAVE_ENTITY="<your-entity>"
export WEAVE_PROJECT_NAME="koog-tracing"
```

## 配置

要啟用 Weave 匯出，請安裝 **OpenTelemetry 功能** 並新增 `WeaveExporter`。該匯出器透過 `OtlpHttpSpanExporter` 使用 Weave 的 OpenTelemetry 端點。

### 範例：帶有 Weave 追蹤的代理程式

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

## 追蹤內容

啟用時，Weave 匯出器會擷取與 Koog 的一般 OpenTelemetry 整合相同的 Span，包括：

-   **代理程式生命週期事件**：代理程式啟動、停止、錯誤
-   **LLM 互動**：提示、完成內容、延遲
-   **工具呼叫**：工具呼叫的執行追蹤
-   **系統上下文**：中繼資料，例如模型名稱、環境、Koog 版本

在 W&B Weave 中視覺化時，追蹤顯示如下：
![W&B Weave traces](img/opentelemetry-weave-exporter-light.png#only-light)
![W&B Weave traces](img/opentelemetry-weave-exporter-dark.png#only-dark)

更多詳情，請參閱官方 [Weave OpenTelemetry 文件](https://weave-docs.wandb.ai/guides/tracking/otel/)。

---

## 疑難排解

### Weave 中沒有出現追蹤
-   確認 `WEAVE_API_KEY`、`WEAVE_ENTITY` 和 `WEAVE_PROJECT_NAME` 已在您的環境中設定。
-   確保您的 W&B 帳戶具有指定實體和專案的存取權限。

### 驗證錯誤
-   檢查您的 `WEAVE_API_KEY` 是否有效。
-   API key 必須具備為所選實體寫入追蹤的權限。

### 連線問題
-   確保您的環境具備對 W&B 的 OpenTelemetry 擷取端點的網路存取。