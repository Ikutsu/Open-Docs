# Ktor 集成：Koog 插件

Koog 自然地融入你的 Ktor 服务器，让你能够使用惯用的 Kotlin API 从两端编写服务器端 AI 应用程序。

只需安装 Koog 插件一次，在 `application.conf`/YAML 或代码中配置你的 LLM 提供商，然后就可以直接从路由中调用代理。不再需要在模块间连接 LLM 客户端——你的路由只需请求一个代理即可开始工作。

## 概述

`koog-ktor` 模块为服务器端代理式开发提供了惯用的 Kotlin/Ktor 集成：

-   即插即用的 Ktor 插件：在你的 Application 中 `install(Koog)`
-   对 OpenAI、Anthropic、Google、OpenRouter、DeepSeek 和 Ollama 的一流支持
-   通过 YAML/CONF 和/或代码进行集中式配置
-   通过提示、工具、特性设置代理；用于路由的简单扩展函数
-   直接 LLM 用法（执行、流式执行、审核）
-   仅限 JVM 的模型上下文协议 (MCP) 工具集成

## 添加依赖项

```kotlin
dependencies {
    implementation("ai.koog:koog-ktor:$koogVersion")
}
```

## 快速开始

1)  配置提供商（在 `application.yaml` 或 `application.conf` 中）

在 `koog.<provider>` 下使用嵌套键。插件会自动识别并加载它们。

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

可选：配置当请求的提供商未配置时，直接 LLM 调用使用的回退。

```yaml
koog:
  llm:
    fallback:
      provider: openai
      # see Model identifiers section below
      model: openai.chat.gpt4_1
```

2)  安装插件并定义路由

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

备注
-   aiAgent 需要一个具体的模型 (LLModel) – 按路由、按用途选择。
-   如需更底层的 LLM 访问，请直接使用 llm() (PromptExecutor)。

## 从路由直接使用 LLM

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

流式传输

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

审核

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

## 以编程方式配置（在代码中）

所有提供商和代理行为都可以通过 `install(Koog) {}` 进行配置。

```kotlin
install(Koog) {
    llm {
        openAI(apiKey = System.getenv("OPENAI_API_KEY") ?: "") {
            baseUrl = "https://api.openai.com"
            timeouts { // Default values shown below
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

        // Optional fallback used by PromptExecutor when a provider isn’t configured
        fallback {
            provider = LLMProvider.OpenAI
            model = OpenAIModels.Chat.GPT4_1
        }
    }

    agentConfig {
        // Provide a reusable base prompt for your agents
        prompt(name = "agent") {
            system("You are a helpful server‑side agent")
        }

        // Limit runaway tools/loops
        maxAgentIterations = 10

        // Register tools available to agents by default
        registerTools {
            // tool(::yourTool) // see Tools Overview for details
        }

        // Install agent features (tracing, etc.)
        // install(OpenTelemetry) { /* ... */ }
    }
}
```

## 配置中的模型标识符（回退）

在 YAML/CONF 中配置 `llm.fallback` 时，请使用这些标识符格式：
-   OpenAI: `openai.chat.gpt4_1`, `openai.reasoning.o3`, `openai.costoptimized.gpt4_1mini`, `openai.audio.gpt4oaudio`, `openai.moderation.omni`
-   Anthropic: `anthropic.sonnet_3_7`, `anthropic.opus_4`, `anthropic.haiku_3_5`
-   Google: `google.gemini2_5pro`, `google.gemini2_0flash001`
-   OpenRouter: `openrouter.gpt4o`, `openrouter.gpt4`, `openrouter.claude3sonnet`
-   DeepSeek: `deepseek.deepseek-chat`, `deepseek.deepseek-reasoner`
-   Ollama: `ollama.meta.llama3.2`, `ollama.alibaba.qwq:32b`, `ollama.groq.llama3-grok-tool-use:8b`

备注
-   对于 OpenAI，你必须包含类别（chat、reasoning、costoptimized、audio、embeddings、moderation）。
-   对于 Ollama，`ollama.model` 和 `ollama.<maker>.<model>` 都受支持。

## MCP 工具（仅限 JVM）

在 JVM 上，你可以将 MCP 服务器中的工具添加到代理工具注册表：

```kotlin
install(Koog) {
    agentConfig {
        mcp {
            // Register via SSE
            sse("https://your-mcp-server.com/sse")

            // Or register via spawned process (stdio transport)
            // process(Runtime.getRuntime().exec("your-mcp-binary ..."))

            // Or from an existing MCP client instance
            // client(existingMcpClient)
        }
    }
}
```

## 为何选择 Koog + Ktor？

-   服务器中代理的 Kotlin 优先、类型安全开发
-   集中式配置，路由代码整洁且可测试
-   为每个路由使用正确的模型，或在直接 LLM 调用时自动回退
-   生产就绪的特性：工具、审核、流式传输和跟踪