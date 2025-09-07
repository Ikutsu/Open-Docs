# Langfuseエクスポーター

Koogは、AIアプリケーションの可観測性と分析のためのプラットフォームである[Langfuse](https://langfuse.com/)へのエージェントトレースのエクスポートを組み込みでサポートしています。
Langfuseとの連携により、KoogエージェントがLLM、API、その他のコンポーネントとどのように対話しているかを視覚化、分析、デバッグできます。

KoogのOpenTelemetryサポートに関する背景情報については、[OpenTelemetryサポート](https://docs.koog.ai/opentelemetry-support/)を参照してください。

---

### セットアップ手順

1.  Langfuseプロジェクトを作成します。[Langfuseで新しいプロジェクトを作成する](https://langfuse.com/docs/get-started#create-new-project-in-langfuse)のセットアップガイドに従ってください。
2.  APIクレデンシャルを取得します。[Langfuse APIキーはどこにありますか？](https://langfuse.com/faq/all/where-are-langfuse-api-keys)で説明されているように、Langfuseの`public key`と`secret key`を取得してください。
3.  Langfuseホスト、公開キー、および秘密キーをLangfuseエクスポーターに渡します。
    これは、`addLangfuseExporter()`関数へのパラメーターとして提供するか、以下に示すように環境変数を設定することによって行えます。

```bash
   export LANGFUSE_HOST="https://cloud.langfuse.com"
   export LANGFUSE_PUBLIC_KEY="<your-public-key>"
   export LANGFUSE_SECRET_KEY="<your-secret-key>"
```

## 設定

Langfuseエクスポートを有効にするには、**OpenTelemetry機能**をインストールし、`LangfuseExporter`を追加します。
エクスポーターは、内部で`OtlpHttpSpanExporter`を使用して、トレースをLangfuseのOpenTelemetryエンドポイントに送信します。

### 例: Langfuseトレースを使用するエージェント

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

## トレースされる内容

有効にすると、LangfuseエクスポーターはKoogの一般的なOpenTelemetry連携と同じスパンをキャプチャします。内訳は以下のとおりです。

-   **エージェントのライフサイクルイベント**: エージェントの開始、停止、エラー
-   **LLMインタラクション**: プロンプト、応答、トークン使用量、レイテンシ
-   **ツール呼び出し**: ツール呼び出しの実行トレース
-   **システムコンテキスト**: モデル名、環境、Koogバージョンなどのメタデータ

Koogはまた、Langfuseが[エージェントグラフ](https://langfuse.com/docs/observability/features/agent-graphs)を表示するために必要なスパン属性もキャプチャします。

Langfuseで視覚化すると、トレースは以下のようになります。
![Langfuse traces](img/opentelemetry-langfuse-exporter-light.png#only-light)
![Langfuse traces](img/opentelemetry-langfuse-exporter-dark.png#only-dark)

LangfuseのOpenTelemetryトレースに関する詳細については、以下を参照してください。
[Langfuse OpenTelemetryドキュメント](https://langfuse.com/integrations/native/opentelemetry#opentelemetry-endpoint)

---

## トラブルシューティング

### Langfuseにトレースが表示されない
-   `LANGFUSE_HOST`、`LANGFUSE_PUBLIC_KEY`、および`LANGFUSE_SECRET_KEY`が環境に設定されていることを再確認してください。
-   セルフホスト型Langfuseで実行している場合は、`LANGFUSE_HOST`がアプリケーション環境から到達可能であることを確認してください。
-   公開/秘密キーのペアが正しいプロジェクトに属していることを確認してください。