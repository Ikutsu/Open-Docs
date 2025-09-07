# W&B Weaveエクスポーター

Koogは、AIアプリケーションの可観測性と分析のためのWeights & Biasesの開発者ツールである[W&B Weave](https://wandb.ai/site/weave/)へのエージェントトレースのエクスポートを組み込みでサポートしています。Weave連携を使用すると、プロンプト、応答、システムコンテキスト、実行トレースをキャプチャし、それらをW&Bワークスペースで直接可視化できます。

KoogのOpenTelemetryサポートに関する背景情報については、[OpenTelemetryサポート](https://docs.koog.ai/opentelemetry-support/)を参照してください。

---

## セットアップ手順

1.  [https://wandb.ai](https://wandb.ai)でW&Bアカウントを作成します。
2.  [https://wandb.ai/authorize](https://wandb.ai/authorize)からAPIキーを取得します。
3.  [https://wandb.ai/home](https://wandb.ai/home)のW&Bダッシュボードにアクセスして、エンティティ (entity) 名を見つけてください。エンティティは通常、個人アカウントの場合はユーザー名、チーム/組織アカウントの場合はチーム名/組織名です。
4.  プロジェクト名を定義します。事前にプロジェクトを作成する必要はありません。最初のトレースが送信されたときに自動的に作成されます。
5.  WeaveエクスポーターにWeaveエンティティ、プロジェクト名、APIキーを渡します。これは、`addWeaveExporter()`関数のパラメーターとして提供するか、または以下に示すように環境変数を設定することで行えます。

```bash
export WEAVE_API_KEY="<your-api-key>"
export WEAVE_ENTITY="<your-entity>"
export WEAVE_PROJECT_NAME="koog-tracing"
```

## 設定

Weaveのエクスポートを有効にするには、**OpenTelemetry機能**をインストールし、`WeaveExporter`を追加します。エクスポーターは、`OtlpHttpSpanExporter`経由でWeaveのOpenTelemetryエンドポイントを使用します。

### 例: Weaveトレースを使用するエージェント

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

## トレースされる内容

有効にすると、WeaveエクスポーターはKoogの一般的なOpenTelemetry連携と同じスパンをキャプチャします。以下を含みます。

-   **エージェントのライフサイクルイベント**: エージェントの開始、停止、エラー
-   **LLMインタラクション**: プロンプト、応答、レイテンシー
-   **ツール呼び出し**: ツール呼び出しの実行トレース
-   **システムコンテキスト**: モデル名、環境、Koogのバージョンなどのメタデータ

W&B Weaveで可視化すると、トレースは以下のようになります。
![W&B Weave traces](img/opentelemetry-weave-exporter-light.png#only-light)
![W&B Weave traces](img/opentelemetry-weave-exporter-dark.png#only-dark)

詳細については、公式の[Weave OpenTelemetry Docs](https://weave-docs.wandb.ai/guides/tracking/otel/)を参照してください。

---

## トラブルシューティング

### Weaveにトレースが表示されない
-   `WEAVE_API_KEY`、`WEAVE_ENTITY`、`WEAVE_PROJECT_NAME`が環境に設定されていることを確認してください。
-   ご使用のW&Bアカウントが、指定されたエンティティとプロジェクトへのアクセス権を持っていることを確認してください。

### 認証エラー
-   `WEAVE_API_KEY`が有効であることを確認してください。
-   APIキーには、選択したエンティティのトレースを書き込む権限が必要です。

### 接続の問題
-   ご使用の環境が、W&BのOpenTelemetry取り込みエンドポイントへのネットワークアクセスを持っていることを確認してください。