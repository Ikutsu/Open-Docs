# Spring Boot連携

Koogは、自動構成スターターを通じてシームレスなSpring Boot連携を提供し、最小限の設定でAIエージェントをSpring Bootアプリケーションに簡単に組み込むことができます。

## 概要

`koog-spring-boot-starter` は、アプリケーションプロパティに基づいてLLMクライアントを自動的に構成し、依存性注入のためにすぐに使えるBeanを提供します。OpenAI、Anthropic、Google、OpenRouter、DeepSeek、Ollamaを含むすべての主要なLLMプロバイダーをサポートしています。

## はじめに

### 1. 依存関係の追加

`build.gradle.kts` にSpring Bootスターターを追加します。

```kotlin
dependencies {
    implementation("ai.koog:koog-spring-boot-starter:$koogVersion")
}
```

### 2. プロバイダーの設定

`application.properties` で優先するLLMプロバイダーを設定します。

```properties
# OpenAIの設定
ai.koog.openai.api-key=${OPENAI_API_KEY}
ai.koog.openai.base-url=https://api.openai.com
# Anthropicの設定  
ai.koog.anthropic.api-key=${ANTHROPIC_API_KEY}
ai.koog.anthropic.base-url=https://api.anthropic.com
# Googleの設定
ai.koog.google.api-key=${GOOGLE_API_KEY}
ai.koog.google.base-url=https://generativelanguage.googleapis.com
# OpenRouterの設定
ai.koog.openrouter.api-key=${OPENROUTER_API_KEY}
ai.koog.openrouter.base-url=https://openrouter.ai
# DeepSeekの設定
ai.koog.deepseek.api-key=${DEEPSEEK_API_KEY}
ai.koog.deepseek.base-url=https://api.deepseek.com
# Ollamaの設定 (ローカル - APIキーは不要)
ai.koog.ollama.base-url=http://localhost:11434
```

またはYAML形式 (`application.yml`) を使用します。

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

!!! tip "環境変数"
APIキーは、セキュリティを確保し、バージョン管理から除外するために、環境変数を使用することを推奨します。

### 3. 注入と使用

自動構成されたExecutorをサービスに注入します。

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

## 高度な使用法

### RESTコントローラーの例

自動構成されたExecutorを使用してチャットエンドポイントを作成する例です。

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

### 複数のプロバイダーのサポート

フォールバックロジックで複数のプロバイダーを処理します。

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
                logger.warn("Executor failed, trying next: ${e.message}") // Executorが失敗しました。次を試します。
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

### 設定プロパティ

カスタムロジックのために設定プロパティを注入することもできます。

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

## 設定リファレンス

### 利用可能なプロパティ

| プロパティ                      | 説明                | Beanの条件                                                      | デフォルト                                  |
|-------------------------------|---------------------|-----------------------------------------------------------------|---------------------------------------------|
| `ai.koog.openai.api-key`      | OpenAI APIキー      | `openAIExecutor` Beanに必須                                     | -                                           |
| `ai.koog.openai.base-url`     | OpenAIベースURL     | オプション                                                      | `https://api.openai.com`                    |
| `ai.koog.anthropic.api-key`   | Anthropic APIキー   | `anthropicExecutor` Beanに必須                                  | -                                           |
| `ai.koog.anthropic.base-url`  | AnthropicベースURL  | オプション                                                      | `https://api.anthropic.com`                 |
| `ai.koog.google.api-key`      | Google APIキー      | `googleExecutor` Beanに必須                                     | -                                           |
| `ai.koog.google.base-url`     | GoogleベースURL     | オプション                                                      | `https://generativelanguage.googleapis.com` |
| `ai.koog.openrouter.api-key`  | OpenRouter APIキー  | `openRouterExecutor` Beanに必須                                 | -                                           |
| `ai.koog.openrouter.base-url` | OpenRouterベースURL | オプション                                                      | `https://openrouter.ai`                     |
| `ai.koog.deepseek.api-key`    | DeepSeek APIキー    | `deepSeekExecutor` Beanに必須                                   | -                                           |
| `ai.koog.deepseek.base-url`   | DeepSeekベースURL   | オプション                                                      | `https://api.deepseek.com`                  |
| `ai.koog.ollama.base-url`     | OllamaベースURL     | いずれかの`ai.koog.ollama.*`プロパティが`ollamaExecutor` Beanをアクティブ化します | `http://localhost:11434`                    |

### Bean名

自動構成は、(設定されている場合に)以下のBeanを作成します。

- `openAIExecutor` - OpenAI Executor (`ai.koog.openai.api-key` が必要)
- `anthropicExecutor` - Anthropic Executor (`ai.koog.anthropic.api-key` が必要)
- `googleExecutor` - Google Executor (`ai.koog.google.api-key` が必要)
- `openRouterExecutor` - OpenRouter Executor (`ai.koog.openrouter.api-key` が必要)
- `deepSeekExecutor` - DeepSeek Executor (`ai.koog.deepseek.api-key` が必要)
- `ollamaExecutor` - Ollama Executor (いずれかの`ai.koog.ollama.*`プロパティが必要)

## トラブルシューティング

### よくある問題

**Beanが見つからないエラー:**

```
No qualifying bean of type 'SingleLLMPromptExecutor' available
```

**解決策:** プロパティファイルに少なくとも1つのプロバイダーが設定されていることを確認してください。

**複数のBeanエラー:**

```
Multiple qualifying beans of type 'SingleLLMPromptExecutor' available
```

**解決策:** `@Qualifier` を使用して、目的のBeanを指定します。

```kotlin
@Service
class MyService(
    @Qualifier("openAIExecutor") private val openAIExecutor: SingleLLMPromptExecutor,
    @Qualifier("anthropicExecutor") private val anthropicExecutor: SingleLLMPromptExecutor
) {
    // ...
}
```

**APIキーがロードされない:**

```
API key is required but not provided
```

**解決策:** 環境変数が正しく設定されており、Spring Bootアプリケーションからアクセスできることを確認してください。

## ベストプラクティス

1.  **環境変数**: APIキーには常に環境変数を使用する
2.  **ヌル許容の注入**: プロバイダーが設定されていないケースを処理するために、ヌル許容型 (`SingleLLMPromptExecutor?`) を使用する
3.  **フォールバックロジック**: 複数のプロバイダーを使用する際にはフォールバックメカニズムを実装する
4.  **エラーハンドリング**: 本番コードではExecutorの呼び出しを常にtry-catchブロックで囲む
5.  **テスト**: 実際のAPI呼び出しを行わないように、テストではモックを使用する
6.  **設定の検証**: Executorを使用する前に、それらが利用可能であることを確認する

## 次のステップ

-   基本的なAIワークフローを構築するために、[シングルランエージェント](single-run-agents.md)について学ぶ
-   高度なユースケースについては、[複雑なワークフローエージェント](complex-workflow-agents.md)を探る
-   エージェントの機能を拡張するために、[ツールの概要](tools-overview.md)を参照する
-   実際の実装については、[例](examples.md)をチェックする
-   フレームワークをよりよく理解するために、[主要な概念](key-concepts.md)を読む