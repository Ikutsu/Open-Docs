# Ktor 整合：Koog 插件

Koog 能自然地整合到您的 Ktor 伺服器中，讓您能使用兩端慣用的 Kotlin API 來編寫伺服器端 AI 應用程式。

只需安裝 Koog 插件一次，在 `application.conf`/YAML 檔中或程式碼中設定您的 LLM 提供者，然後就可以直接從您的路由呼叫 agents。不再需要在模組之間連接 LLM 客戶端 – 您的路由只需請求一個 agent 即可立即使用。

## 概述

`koog-ktor` 模組為伺服器端 agent 開發提供了慣用的 Kotlin/Ktor 整合：

- 隨插即用的 Ktor 插件：在您的 `Application` 中 `install(Koog)`
- 對 OpenAI、Anthropic、Google、OpenRouter、DeepSeek 和 Ollama 的一流支援
- 透過 YAML/CONF 和/或程式碼進行集中式配置
- 帶有 prompt、工具、功能的 Agent 設定；針對路由的簡單擴展函數
- 直接 LLM 使用（`execute`、`executeStreaming`、`moderate`）
- 僅限 JVM 的 Model Context Protocol (MCP) 工具整合

## 新增依賴項

```kotlin
dependencies {
    implementation("ai.koog:koog-ktor:$koogVersion")
}
```

## 快速入門

1) 配置提供者（在 `application.yaml` 或 `application.conf` 中）

使用 `koog.<provider>` 下的巢狀鍵。插件會自動識別它們。

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
  # Ollama is enabled when any koog.ollama.* key exists
  ollama:
    enable: true
    baseUrl: http://localhost:11434
```

選用：配置當請求的提供者未配置時，直接 LLM 呼叫所使用的 fallback。

```yaml
koog:
  llm:
    fallback:
      provider: openai
      # see Model identifiers section below
      model: openai.chat.gpt4_1
```

2) 安裝插件並定義路由

```kotlin
fun Application.module() {
    install(Koog) {
        // You can also configure providers programmatically (see below)
    }

    routing {
        route("/ai") {
            post("/chat") {
                val userInput = call.receiveText()
                // Create and run a default single‑run agent using a specific model
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

附註
- `aiAgent` 需要一個具體的模型 (`LLModel`) – 依路由、依用途選擇。
- 對於較低層級的 LLM 存取，直接使用 `llm()` (`PromptExecutor`)。

## 從路由直接使用 LLM

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

    // Join all assistant messages into a single string
    val text = messages.joinToString(separator = "") { it.content }
    call.respond(HttpStatusCode.OK, text)
}
```

串流

```kotlin
get("/stream") {
    val flow = llm().executeStreaming(
        prompt("streaming") { user("Stream this response, please") },
        OpenRouterModels.GPT4o
    )

    // Example: buffer and send as one chunk
    val sb = StringBuilder()
    flow.collect { chunk -> sb.append(chunk) }
    call.respondText(sb.toString())
}
```

內容審核

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

## 程式碼式配置（在程式碼中）

所有提供者和 agent 行為都可以透過 `install(Koog) {}` 進行配置。

```kotlin
install(Koog) {
    llm {
        openAI(apiKey = System.getenv("OPENAI_API_KEY") ?: "") {
            baseUrl = "https://api.openai.com"
            timeouts { // 預設值如下所示
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

        // 選用：當提供者未配置時，PromptExecutor 使用的 fallback
        fallback {
            provider = LLMProvider.OpenAI
            model = OpenAIModels.Chat.GPT4_1
        }
    }

    agentConfig {
        // 為您的 agents 提供一個可重複使用的基礎 prompt
        prompt(name = "agent") {
            system("您是一個有用的伺服器端 agent")
        }

        // 限制失控的工具/迴圈
        maxAgentIterations = 10

        // 預設為 agents 註冊可用的工具
        registerTools {
            // tool(::yourTool) // 請參閱 Tools Overview 以獲取詳細資訊
        }

        // 安裝 agent 功能（追蹤等）
        // install(OpenTelemetry) { /* ... */ }
    }
}
```

## 配置中的模型識別碼（fallback）

在 YAML/CONF 中配置 `llm.fallback` 時，請使用這些識別碼格式：
- OpenAI: `openai.chat.gpt4_1`, `openai.reasoning.o3`, `openai.costoptimized.gpt4_1mini`, `openai.audio.gpt4oaudio`, `openai.moderation.omni`
- Anthropic: `anthropic.sonnet_3_7`, `anthropic.opus_4`, `anthropic.haiku_3_5`
- Google: `google.gemini2_5pro`, `google.gemini2_0flash001`
- OpenRouter: `openrouter.gpt4o`, `openrouter.gpt4`, `openrouter.claude3sonnet`
- DeepSeek: `deepseek.deepseek-chat`, `deepseek.deepseek-reasoner`
- Ollama: `ollama.meta.llama3.2`, `ollama.alibaba.qwq:32b`, `ollama.groq.llama3-grok-tool-use:8b`

附註
- 對於 OpenAI，您必須包含類別（`chat`、`reasoning`、`costoptimized`、`audio`、`embeddings`、`moderation`）。
- 對於 Ollama，`ollama.model` 和 `ollama.<maker>.<model>` 都受支援。

## MCP 工具（僅限 JVM）

在 JVM 上，您可以將來自 MCP 伺服器的工具新增到您的 agent 工具註冊中心：

```kotlin
install(Koog) {
    agentConfig {
        mcp {
            // 透過 SSE 註冊
            sse("https://your-mcp-server.com/sse")

            // 或者透過生成程序（stdio 傳輸）註冊
            // process(Runtime.getRuntime().exec("your-mcp-binary ..."))

            // 或者從現有的 MCP 客戶端實例註冊
            // client(existingMcpClient)
        }
    }
}
```

## 為何選擇 Koog + Ktor？

- 在您的伺服器中，優先使用 Kotlin 進行型別安全的 agent 開發
- 集中式配置，搭配簡潔、可測試的路由程式碼
- 針對每個路由使用正確的模型，或在直接 LLM 呼叫時自動 fallback
- 生產就緒的功能：工具、內容審核、串流和追蹤