# Spring Boot 整合

Koog 透過其自動配置啟動器提供無縫的 Spring Boot 整合，讓您可以輕鬆地將 AI 代理整合到您的 Spring Boot 應用程式中，只需最少的設定。

## 概觀

`koog-spring-boot-starter` 會根據您的應用程式屬性自動配置 LLM 客戶端，並提供現成的 Bean 以供依賴注入使用。它支援所有主要的 LLM 供應商，包括 OpenAI、Anthropic、Google、OpenRouter、DeepSeek 和 Ollama。

## 開始使用

### 1. 新增依賴

將 Spring Boot 啟動器新增到您的 `build.gradle.kts`：

```kotlin
dependencies {
    implementation("ai.koog:koog-spring-boot-starter:$koogVersion")
}
```

### 2. 配置供應商

在 `application.properties` 中配置您偏好的 LLM 供應商：

```properties
# OpenAI Configuration
ai.koog.openai.api-key=${OPENAI_API_KEY}
ai.koog.openai.base-url=https://api.openai.com
# Anthropic Configuration  
ai.koog.anthropic.api-key=${ANTHROPIC_API_KEY}
ai.koog.anthropic.base-url=https://api.anthropic.com
# Google Configuration
ai.koog.google.api-key=${GOOGLE_API_KEY}
ai.koog.google.base-url=https://generativelanguage.googleapis.com
# OpenRouter Configuration
ai.koog.openrouter.api-key=${OPENROUTER_API_KEY}
ai.koog.openrouter.base-url=https://openrouter.ai
# DeepSeek Configuration
ai.koog.deepseek.api-key=${DEEPSEEK_API_KEY}
ai.koog.deepseek.base-url=https://api.deepseek.com
# Ollama Configuration (local - no API key required)
ai.koog.ollama.base-url=http://localhost:11434
```

或使用 YAML 格式 (`application.yml`)：

```yaml
ai:
    koog:
        openai:
            api-key: ${OPENAI_API_KEY}
            base-url: https://api.openai.com
        anthropic:
            api-key: ${ANTHROPIC_API_KEY}
            base-url: https://api.anthropic.com
        google:
            api-key: ${GOOGLE_API_KEY}
            base-url: https://generativelanguage.googleapis.com
        openrouter:
            api-key: ${OPENROUTER_API_KEY}
            base-url: https://openrouter.ai
        deepseek:
            api-key: ${DEEPSEEK_API_KEY}
            base-url: https://api.deepseek.com
        ollama:
            base-url: http://localhost:11434
```

!!! tip "環境變數"
建議使用環境變數來存放 API 金鑰，以確保它們安全並避免被納入版本控制。

### 3. 注入與使用

將自動配置的執行器注入到您的服務中：

```kotlin
@Service
class AIService(
    private val openAIExecutor: SingleLLMPromptExecutor?,
    private val anthropicExecutor: SingleLLMPromptExecutor?
) {

    suspend fun generateResponse(input: String): String {
        val prompt = prompt {
            system("You are a helpful AI assistant")
            user(input)
        }

        return when {
            openAIExecutor != null -> {
                val result = openAIExecutor.execute(prompt)
                result.text
            }
            anthropicExecutor != null -> {
                val result = anthropicExecutor.execute(prompt)
                result.text
            }
            else -> throw IllegalStateException("No LLM provider configured")
        }
    }
}
```

## 進階使用

### REST 控制器範例

使用自動配置的執行器建立一個聊天端點：

```kotlin
@RestController
@RequestMapping("/api/chat")
class ChatController(
    private val anthropicExecutor: SingleLLMPromptExecutor?
) {

    @PostMapping
    suspend fun chat(@RequestBody request: ChatRequest): ResponseEntity<ChatResponse> {
        return if (anthropicExecutor != null) {
            try {
                val prompt = prompt {
                    system("You are a helpful assistant")
                    user(request.message)
                }

                val result = anthropicExecutor.execute(prompt)
                ResponseEntity.ok(ChatResponse(result.text))
            } catch (e: Exception) {
                ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body(ChatResponse("Error processing request"))
            }
        } else {
            ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE)
                .body(ChatResponse("AI service not configured"))
        }
    }
}

data class ChatRequest(val message: String)
data class ChatResponse(val response: String)
```

### 多供應商支援

處理多個供應商並提供回退邏輯：

```kotlin
@Service
class RobustAIService(
    private val openAIExecutor: SingleLLMPromptExecutor?,
    private val anthropicExecutor: SingleLLMPromptExecutor?,
    private val openRouterExecutor: SingleLLMPromptExecutor?
) {

    suspend fun generateWithFallback(input: String): String {
        val prompt = prompt {
            system("You are a helpful AI assistant")
            user(input)
        }

        val executors = listOfNotNull(openAIExecutor, anthropicExecutor, openRouterExecutor)

        for (executor in executors) {
            try {
                val result = executor.execute(prompt)
                return result.text
            } catch (e: Exception) {
                logger.warn("Executor failed, trying next: ${e.message}")
                continue
            }
        }

        throw IllegalStateException("All AI providers failed")
    }

    companion object {
        private val logger = LoggerFactory.getLogger(RobustAIService::class.java)
    }
}
```

### 配置屬性

您還可以注入配置屬性以實現自訂邏輯：

```kotlin
@Service
class ConfigurableAIService(
    private val openAIExecutor: SingleLLMPromptExecutor?,
    @Value("\${ai.koog.openai.api-key:}") private val openAIKey: String
) {

    fun isOpenAIConfigured(): Boolean = openAIKey.isNotBlank() && openAIExecutor != null

    suspend fun processIfConfigured(input: String): String? {
        return if (isOpenAIConfigured()) {
            val result = openAIExecutor!!.execute(prompt { user(input) })
            result.text
        } else {
            null
        }
    }
}
```

## 配置參考

### 可用屬性

| Property                      | 描述                  | Bean 條件                                                   | 預設值                                      |
|-------------------------------|-----------------------|-------------------------------------------------------------|---------------------------------------------|
| `ai.koog.openai.api-key`      | OpenAI API 金鑰       | `openAIExecutor` bean 所必需                                | -                                           |
| `ai.koog.openai.base-url`     | OpenAI 基礎 URL       | 選填                                                        | `https://api.openai.com`                    |
| `ai.koog.anthropic.api-key`   | Anthropic API 金鑰    | `anthropicExecutor` bean 所必需                             | -                                           |
| `ai.koog.anthropic.base-url`  | Anthropic 基礎 URL    | 選填                                                        | `https://api.anthropic.com`                 |
| `ai.koog.google.api-key`      | Google API 金鑰       | `googleExecutor` bean 所必需                                | -                                           |
| `ai.koog.google.base-url`     | Google 基礎 URL       | 選填                                                        | `https://generativelanguage.googleapis.com` |
| `ai.koog.openrouter.api-key`  | OpenRouter API 金鑰   | `openRouterExecutor` bean 所必需                            | -                                           |
| `ai.koog.openrouter.base-url` | OpenRouter 基礎 URL   | 選填                                                        | `https://openrouter.ai`                     |
| `ai.koog.deepseek.api-key`    | DeepSeek API 金鑰     | `deepSeekExecutor` bean 所必需                              | -                                           |
| `ai.koog.deepseek.base-url`   | DeepSeek 基礎 URL     | 選填                                                        | `https://api.deepseek.com`                  |
| `ai.koog.ollama.base-url`     | Ollama 基礎 URL       | 任何 `ai.koog.ollama.*` 屬性都會啟用 `ollamaExecutor` bean | `http://localhost:11434`                    |

### Bean 名稱

自動配置會建立以下 Bean（當配置後）：

- `openAIExecutor` - OpenAI 執行器（需要 `ai.koog.openai.api-key`）
- `anthropicExecutor` - Anthropic 執行器（需要 `ai.koog.anthropic.api-key`）
- `googleExecutor` - Google 執行器（需要 `ai.koog.google.api-key`）
- `openRouterExecutor` - OpenRouter 執行器（需要 `ai.koog.openrouter.api-key`）
- `deepSeekExecutor` - DeepSeek 執行器（需要 `ai.koog.deepseek.api-key`）
- `ollamaExecutor` - Ollama 執行器（需要任何 `ai.koog.ollama.*` 屬性）

## 故障排除

### 常見問題

**找不到 Bean 錯誤：**

```
No qualifying bean of type 'SingleLLMPromptExecutor' available
```

**解決方案：** 請確保您已在屬性檔案中配置至少一個供應商。

**多個 Bean 錯誤：**

```
Multiple qualifying beans of type 'SingleLLMPromptExecutor' available
```

**解決方案：** 使用 `@Qualifier` 來指定您想要的 Bean：

```kotlin
@Service
class MyService(
    @Qualifier("openAIExecutor") private val openAIExecutor: SingleLLMPromptExecutor,
    @Qualifier("anthropicExecutor") private val anthropicExecutor: SingleLLMPromptExecutor
) {
    // ...
}
```

**API 金鑰未載入：**

```
API key is required but not provided
```

**解決方案：** 檢查您的環境變數是否已正確設定並可供您的 Spring Boot 應用程式存取。

## 最佳實踐

1. **環境變數**：始終使用環境變數來存放 API 金鑰
2. **可為空的注入**：使用可為空值類型 (`SingleLLMPromptExecutor?`) 來處理供應商未配置的情況
3. **回退邏輯**：在使用多個供應商時實作回退機制
4. **錯誤處理**：在生產程式碼中，始終將執行器呼叫包裝在 try-catch 區塊中
5. **測試**：在測試中使用模擬物件以避免實際的 API 呼叫
6. **配置驗證**：在使用執行器之前檢查它們是否可用

## 後續步驟

- 了解 [單次執行代理](single-run-agents.md) 以建立基本的 AI 工作流程
- 探索 [複雜工作流程代理](complex-workflow-agents.md) 以用於進階使用情境
- 查看 [工具概述](tools-overview.md) 以擴展您代理的功能
- 查閱 [範例](examples.md) 以了解實際應用
- 閱讀 [核心概念](key-concepts.md) 以更好地理解此框架