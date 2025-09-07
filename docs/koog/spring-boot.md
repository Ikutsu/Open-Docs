# Spring Boot 整合

Koog 通过其自动配置 starter，提供了与 Spring Boot 的无缝整合，使得将 AI 代理轻松纳入您的 Spring Boot 应用程序变得简单，只需最少的设置。

## 概述

`koog-spring-boot-starter` 根据您的应用程序属性自动配置 LLM 客户端，并提供即用型 bean 以进行依赖注入。它支持所有主流 LLM 提供商，包括 OpenAI、Anthropic、Google、OpenRouter、DeepSeek 和 Ollama。

## 快速入门

### 1. 添加依赖项

将 Spring Boot starter 添加到您的 `build.gradle.kts`：

```kotlin
dependencies {
    implementation("ai.koog:koog-spring-boot-starter:$koogVersion")
}
```

### 2. 配置提供商

在 `application.properties` 中配置您首选的 LLM 提供商：

```properties
# OpenAI 配置
ai.koog.openai.api-key=${OPENAI_API_KEY}
ai.koog.openai.base-url=https://api.openai.com
# Anthropic 配置  
ai.koog.anthropic.api-key=${ANTHROPIC_API_KEY}
ai.koog.anthropic.base-url=https://api.anthropic.com
# Google 配置
ai.koog.google.api-key=${GOOGLE_API_KEY}
ai.koog.google.base-url=https://generativelanguage.googleapis.com
# OpenRouter 配置
ai.koog.openrouter.api-key=${OPENROUTER_API_KEY}
ai.koog.openrouter.base-url=https://openrouter.ai
# DeepSeek 配置
ai.koog.deepseek.api-key=${DEEPSEEK_API_KEY}
ai.koog.deepseek.base-url=https://api.deepseek.com
# Ollama 配置（本地 - 无需 API 密钥）
ai.koog.ollama.base-url=http://localhost:11434
```

或使用 YAML 格式（`application.yml`）：

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

!!! tip "环境变量"
建议使用环境变量来存储 API 密钥，以确保其安全并避免纳入版本控制。

### 3. 注入与使用

将自动配置的执行器注入您的服务：

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

## 高级用法

### REST 控制器示例

使用自动配置的执行器创建聊天端点：

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

### 多提供商支持

处理带有回退逻辑的多个提供商：

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
                logger.warn("执行器失败，尝试下一个：${e.message}")
                continue
            }
        }

        throw IllegalStateException("所有 AI 提供商都失败了")
    }

    companion object {
        private val logger = LoggerFactory.getLogger(RobustAIService::class.java)
    }
}
```

### 配置属性

您也可以注入配置属性以实现自定义逻辑：

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

## 配置参考

### 可用属性

| 属性                            | 描述              | Bean 条件                                                  | 默认值                                     |
|-------------------------------|-------------------|--------------------------------------------------------|---------------------------------------------|
| `ai.koog.openai.api-key`      | OpenAI API 密钥     | `openAIExecutor` bean 所必需                           | -                                           |
| `ai.koog.openai.base-url`     | OpenAI 基础 URL     | 可选                                                         | `https://api.openai.com`                    |
| `ai.koog.anthropic.api-key`   | Anthropic API 密钥  | `anthropicExecutor` bean 所必需                        | -                                           |
| `ai.koog.anthropic.base-url`  | Anthropic 基础 URL  | 可选                                                         | `https://api.anthropic.com`                 |
| `ai.koog.google.api-key`      | Google API 密钥     | `googleExecutor` bean 所必需                           | -                                           |
| `ai.koog.google.base-url`     | Google 基础 URL     | 可选                                                         | `https://generativelanguage.googleapis.com` |
| `ai.koog.openrouter.api-key`  | OpenRouter API 密钥 | `openRouterExecutor` bean 所必需                       | -                                           |
| `ai.koog.openrouter.base-url` | OpenRouter 基础 URL | 可选                                                         | `https://openrouter.ai`                     |
| `ai.koog.deepseek.api-key`    | DeepSeek API 密钥   | `deepSeekExecutor` bean 所必需                         | -                                           |
| `ai.koog.deepseek.base-url`   | DeepSeek 基础 URL   | 可选                                                         | `https://api.deepseek.com`                  |
| `ai.koog.ollama.base-url`     | Ollama 基础 URL     | 任何 `ai.koog.ollama.*` 属性都会激活 `ollamaExecutor` bean | `http://localhost:11434`                    |

### Bean 名称

自动配置会创建以下 bean（当配置时）：

- `openAIExecutor` - OpenAI 执行器（需要 `ai.koog.openai.api-key`）
- `anthropicExecutor` - Anthropic 执行器（需要 `ai.koog.anthropic.api-key`）
- `googleExecutor` - Google 执行器（需要 `ai.koog.google.api-key`）
- `openRouterExecutor` - OpenRouter 执行器（需要 `ai.koog.openrouter.api-key`）
- `deepSeekExecutor` - DeepSeek 执行器（需要 `ai.koog.deepseek.api-key`）
- `ollamaExecutor` - Ollama 执行器（需要任何 `ai.koog.ollama.*` 属性）

## 故障排除

### 常见问题

**Bean 未找到错误：**

```
No qualifying bean of type 'SingleLLMPromptExecutor' available
```

**解决方案：** 请确保您已在属性文件中配置了至少一个提供商。

**多个 bean 错误：**

```
Multiple qualifying beans of type 'SingleLLMPromptExecutor' available
```

**解决方案：** 使用 `@Qualifier` 来指定您想要的 bean：

```kotlin
@Service
class MyService(
    @Qualifier("openAIExecutor") private val openAIExecutor: SingleLLMPromptExecutor,
    @Qualifier("anthropicExecutor") private val anthropicExecutor: SingleLLMPromptExecutor
) {
    // ...
}
```

**API 密钥未加载：**

```
API key is required but not provided
```

**解决方案：** 请检查您的环境变量是否已正确设置并可供您的 Spring Boot 应用程序访问。

## 最佳实践

1.  **环境变量**：始终使用环境变量来存储 API 密钥
2.  **可空注入**：使用可空类型（`SingleLLMPromptExecutor?`）来处理提供商未配置的情况
3.  **回退逻辑**：在使用多个提供商时实现回退机制
4.  **错误处理**：始终将执行器调用包装在 try-catch 块中，以用于生产代码
5.  **测试**：在测试中使用模拟对象以避免进行实际的 API 调用
6.  **配置验证**：在使用执行器之前检查其是否可用

## 下一步

- 了解 [单次运行代理](single-run-agents.md) 以构建基本的 AI 工作流
- 探索 [复杂工作流代理](complex-workflow-agents.md) 以应对高级用例
- 请参见 [工具概述](tools-overview.md) 以扩展您的代理功能
- 查看 [示例](examples.md) 以了解实际实现
- 阅读 [关键概念](key-concepts.md) 以更好地理解该框架