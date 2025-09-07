# Ktor連携: Koogプラグイン

KoogはKtorサーバーに自然に統合され、双方からKotlinらしいAPIを使用して、サーバーサイドのAIアプリケーションを記述できます。

Koogプラグインを一度インストールし、`application.conf`またはYAML、あるいはコード内でLLMプロバイダーを設定すれば、ルートから直接エージェントを呼び出せます。モジュールをまたいでLLMクライアントを配線する必要はもうありません。ルートはエージェントを要求するだけで準備完了です。

## 概要

`koog-ktor`モジュールは、サーバーサイドのエージェント開発向けに、Kotlin/Ktorに即した統合機能を提供します。

- 組み込み可能なKtorプラグイン: `Application`内で`install(Koog)`
- OpenAI、Anthropic、Google、OpenRouter、DeepSeek、Ollamaのファーストクラスのサポート
- YAML/CONFまたはコード、あるいはその両方による集中型設定
- プロンプト、ツール、機能を備えたエージェントのセットアップ。ルート用のシンプルな拡張関数
- LLMの直接利用 (`execute`, `executeStreaming`, `moderate`)
- JVM専用のModel Context Protocol (MCP) ツール統合

## 依存関係の追加

```kotlin
dependencies {
    implementation("ai.koog:koog-ktor:$koogVersion")
}
```

## クイックスタート

1) プロバイダーの設定 (`application.yaml` または `application.conf`内)

`koog.<provider>`の下にネストされたキーを使用します。プラグインがそれらを自動的に認識します。

```yaml
# application.yaml (Ktor config)
koog:
  openai:
    apikey: ${OPENAI_API_KEY}
    baseUrl: https://api.openai.com
  anthropic:
    apikey: ${ANTHROPIC_API_KEY}
    baseUrl: https://api.anthropic.com
  google:
    apikey: ${GOOGLE_API_KEY}
    baseUrl: https://generativelanguage.googleapis.com
  openrouter:
    apikey: ${OPENROUTER_API_KEY}
    baseUrl: https://openrouter.ai
  deepseek:
    apikey: ${DEEPSEEK_API_KEY}
    baseUrl: https://api.deepseek.com
  # koog.ollama.* のいずれかのキーが存在する場合、Ollamaが有効になります
  ollama:
    enable: true
    baseUrl: http://localhost:11434
```

オプション: 要求されたプロバイダーが設定されていない場合に、LLMの直接呼び出しで使用されるフォールバックを設定します。

```yaml
koog:
  llm:
    fallback:
      provider: openai
      # 以下の「モデル識別子」セクションを参照
      model: openai.chat.gpt4_1
```

2) プラグインをインストールし、ルートを定義する

```kotlin
fun Application.module() {
    install(Koog) {
        // プロバイダーはプログラムからも設定できます (以下を参照)
    }

    routing {
        route("/ai") {
            post("/chat") {
                val userInput = call.receiveText()
                // 特定のモデルを使用して、デフォルトの単一実行エージェントを作成し、実行する
                val output = aiAgent(
                    strategy = reActStrategy(),
                    model = OpenAIModels.Chat.GPT4_1,
                    input = userInput
                )
                call.respond(HttpStatusCode.OK, output)
            }
        }
    }
}
```

注意
- `aiAgent`は具体的なモデル (`LLModel`) を必要とします — ルートごと、使用ごとに選択してください。
- より低レベルなLLMアクセスには、`llm()` (`PromptExecutor`) を直接使用してください。

## ルートからのLLM直接利用

```kotlin
post("/llm-chat") {
    val userInput = call.receiveText()

    val messages = llm().execute(
        prompt("chat") {
            system("You are a helpful assistant that clarifies questions")
            user(userInput)
        },
        GoogleModels.Gemini2_5Pro
    )

    // すべてのアシスタントメッセージを1つの文字列に結合する
    val text = messages.joinToString(separator = "") { it.content }
    call.respond(HttpStatusCode.OK, text)
}
```

ストリーミング

```kotlin
get("/stream") {
    val flow = llm().executeStreaming(
        prompt("streaming") { user("Stream this response, please") },
        OpenRouterModels.GPT4o
    )

    // 例: バッファリングして1つのチャンクとして送信する
    val sb = StringBuilder()
    flow.collect { chunk -> sb.append(chunk) }
    call.respondText(sb.toString())
}
```

モデレーション

```kotlin
post("/moderated-chat") {
    val userInput = call.receiveText()

    val moderation = llm().moderate(
        prompt("moderation") { user(userInput) },
        OpenAIModels.Moderation.Omni
    )

    if (moderation.isHarmful) {
        call.respond(HttpStatusCode.BadRequest, "Harmful content detected")
        return@post
    }

    val output = aiAgent(
        strategy = reActStrategy(),
        model = OpenAIModels.Chat.GPT4_1,
        input = userInput
    )
    call.respond(HttpStatusCode.OK, output)
}
```

## プログラムによる設定 (コード内)

すべてのプロバイダーとエージェントの動作は `install(Koog) {}` を通じて設定できます。

```kotlin
install(Koog) {
    llm {
        openAI(apiKey = System.getenv("OPENAI_API_KEY") ?: "") {
            baseUrl = "https://api.openai.com"
            timeouts { // 以下にデフォルト値を示します
                requestTimeout = 15.minutes
                connectTimeout = 60.seconds
                socketTimeout = 15.minutes
            }
        }
        anthropic(apiKey = System.getenv("ANTHROPIC_API_KEY") ?: "")
        google(apiKey = System.getenv("GOOGLE_API_KEY") ?: "")
        openRouter(apiKey = System.getenv("OPENROUTER_API_KEY") ?: "")
        deepSeek(apiKey = System.getenv("DEEPSEEK_API_KEY") ?: "")
        ollama { baseUrl = "http://localhost:11434" }

        // プロバイダーが設定されていない場合に PromptExecutor が使用するオプションのフォールバック
        fallback {
            provider = LLMProvider.OpenAI
            model = OpenAIModels.Chat.GPT4_1
        }
    }

    agentConfig {
        // エージェント用の再利用可能なベースプロンプトを提供する
        prompt(name = "agent") {
            system("You are a helpful server‑side agent")
        }

        // 暴走するツール/ループを制限する
        maxAgentIterations = 10

        // エージェントがデフォルトで利用できるツールを登録する
        registerTools {
            // tool(::yourTool) // 詳細はツール概要を参照
        }

        // エージェント機能 (トレーシングなど) をインストールする
        // install(OpenTelemetry) { /* ... */ }
    }
}
```

## 設定内のモデル識別子 (フォールバック)

YAML/CONFで `llm.fallback` を設定する際は、以下の識別子形式を使用してください。
- OpenAI: openai.chat.gpt4_1, openai.reasoning.o3, openai.costoptimized.gpt4_1mini, openai.audio.gpt4oaudio, openai.moderation.omni
- Anthropic: anthropic.sonnet_3_7, anthropic.opus_4, anthropic.haiku_3_5
- Google: google.gemini2_5pro, google.gemini2_0flash001
- OpenRouter: openrouter.gpt4o, openrouter.gpt4, openrouter.claude3sonnet
- DeepSeek: deepseek.deepseek-chat, deepseek.deepseek-reasoner
- Ollama: ollama.meta.llama3.2, ollama.alibaba.qwq:32b, ollama.groq.llama3-grok-tool-use:8b

注意
- OpenAIの場合、カテゴリ (`chat`, `reasoning`, `costoptimized`, `audio`, `embeddings`, `moderation`) を含める必要があります。
- Ollamaの場合、`ollama.model` と `ollama.<maker>.<model>` の両方がサポートされています。

## MCPツール (JVM専用)

JVMでは、MCPサーバーからツールをエージェントのツールレジストリに追加できます。

```kotlin
install(Koog) {
    agentConfig {
        mcp {
            // SSE経由で登録
            sse("https://your-mcp-server.com/sse")

            // または、生成されたプロセス (標準入出力転送) 経由で登録
            // process(Runtime.getRuntime().exec("your-mcp-binary ..."))

            // または、既存のMCPクライアントインスタンスから
            // client(existingMcpClient)
        }
    }
}
```
## Koog + Ktorを選ぶ理由

- サーバーでのKotlinファーストな型安全なエージェント開発
- クリーンでテストしやすいルートコードによる集中型設定
- ルートごとに適切なモデルを使用するか、LLMの直接呼び出しには自動的にフォールバックする
- 本番環境対応の機能: ツール、モデレーション、ストリーミング、トレーシング