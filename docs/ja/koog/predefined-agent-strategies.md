# 事前定義されたエージェント戦略

エージェントの実装を容易にするため、Koogは一般的なエージェントのユースケース向けに事前定義されたエージェント戦略を提供します。
以下の事前定義された戦略が利用可能です。

- [チャットエージェント戦略](#chat-agent-strategy)
- [ReAct戦略](#react-strategy)

## チャットエージェント戦略

チャットエージェント戦略は、チャット対話プロセスを実行するために設計されています。
これは、ユーザー入力の処理、ツールの実行、チャットのような応答の提供を行うために、異なるステージ、ノード、ツールの間の相互作用を調整します。

### 概要

チャットエージェント戦略は、エージェントが次のパターンを実装します。

1.  ユーザー入力を受け取る
2.  LLMを使用して入力を処理する
3.  ツールを呼び出すか、直接応答を提供する
4.  ツールの結果を処理し、会話を続行する
5.  LLMがツールを使用せずにプレーンテキストで応答しようとした場合にフィードバックを提供する

このアプローチにより、エージェントがツールを使用してユーザーの要求に応えることができる会話型インターフェースが作成されます。

### セットアップと依存関係

Koogにおけるチャットエージェント戦略の実装は、`chatAgentStrategy`関数を介して行われます。この関数をエージェントコードで利用できるようにするには、次の依存関係インポートを追加してください。

```
ai.koog.agents.ext.agent.chatAgentStrategy
```

この戦略を使用するには、以下のパターンに従ってAIエージェントを作成します。

<!--- INCLUDE
import ai.koog.agents.core.agent.AIAgent
import ai.koog.prompt.executor.llms.all.simpleOpenAIExecutor
import ai.koog.agents.ext.agent.chatAgentStrategy
import ai.koog.agents.core.tools.ToolRegistry
import ai.koog.prompt.executor.clients.openai.OpenAIModels

val apiKey = System.getenv("OPENAI_API_KEY") ?: error("Please set OPENAI_API_KEY environment variable")
val promptExecutor = simpleOpenAIExecutor(apiKey)
val toolRegistry = ToolRegistry.EMPTY
val model =  OpenAIModels.Reasoning.O4Mini
-->
```kotlin
val chatAgent = AIAgent(
    executor = promptExecutor,
    toolRegistry = toolRegistry,
    llmModel = model,
    // chatAgentStrategy をエージェント戦略として設定
    strategy = chatAgentStrategy()
)
```
<!--- KNIT example-predefined-strategies-01.kt -->

### チャットエージェント戦略を使用するタイミング

チャットエージェント戦略は、特に次の用途に役立ちます。

-   ツールを使用する必要がある会話型エージェントを構築する
-   ユーザー要求に基づいてアクションを実行できるアシスタントを作成する
-   外部システムやデータにアクセスする必要があるチャットボットを実装する
-   プレーンテキストの応答ではなく、ツール使用を強制したいシナリオ

### 例

ここに、事前定義されたチャットエージェント戦略（`chatAgentStrategy`）とエージェントが使用する可能性のあるツールを実装するAIエージェントのコード例を示します。

<!--- INCLUDE
import ai.koog.agents.core.agent.AIAgent
import ai.koog.prompt.executor.llms.all.simpleOpenAIExecutor
import ai.koog.agents.ext.agent.chatAgentStrategy
import ai.koog.agents.core.tools.ToolRegistry
import ai.koog.prompt.executor.clients.openai.OpenAIModels
import ai.koog.agents.ext.tool.AskUser
import ai.koog.agents.ext.tool.SayToUser

typealias searchTool = AskUser
typealias weatherTool = SayToUser

val apiKey = System.getenv("OPENAI_API_KEY") ?: error("Please set OPENAI_API_KEY environment variable")
val promptExecutor = simpleOpenAIExecutor(apiKey)
val toolRegistry = ToolRegistry.EMPTY
val model =  OpenAIModels.Reasoning.O4Mini
-->
```kotlin
val chatAgent = AIAgent(
    executor = promptExecutor,
    llmModel = model,
    // chatAgentStrategy をエージェント戦略として使用
    strategy = chatAgentStrategy(),
    // エージェントが使用できるツールを追加
    toolRegistry = ToolRegistry {
        tool(searchTool)
        tool(weatherTool)
    }
)

suspend fun main() { 
    // ユーザーのクエリでエージェントを実行
    val result = chatAgent.run("What's the weather like today and should I bring an umbrella?")
}
```
<!--- KNIT example-predefined-strategies-02.kt -->

## ReAct戦略

ReAct（Reasoning and Acting）戦略は、推論と実行のステージを交互に行い、タスクを動的に処理し、大規模言語モデル（LLM）からの出力を要求するAIエージェント戦略です。

### 概要

ReAct戦略は、エージェントが次のパターンを実装します。

1.  現在の状態について推論し、次のステップを計画する
2.  その推論に基づいて行動を起こす
3.  それらの行動の結果を観察する
4.  このサイクルを繰り返す

このアプローチは、推論（問題を段階的に考えること）と行動（情報を収集したり操作を実行するためにツールを実行すること）の強みを組み合わせます。

### フロー図

ここにReAct戦略のフロー図を示します。

![Koog flow diagram](img/koog-react-diagram-light.png#only-light)
![Koog flow diagram](img/koog-react-diagram-dark.png#only-dark)

### セットアップと依存関係

KoogにおけるReAct戦略の実装は、`reActStrategy`関数を介して行われます。この関数をエージェントコードで利用できるようにするには、次の依存関係インポートを追加してください。

```
ai.koog.agents.ext.agent.reActStrategy
```
<!--- KNIT example-predefined-strategies-03.kt -->

この戦略を使用するには、以下のパターンに従ってAIエージェントを作成します。

<!--- INCLUDE
import ai.koog.agents.core.agent.AIAgent
import ai.koog.prompt.executor.llms.all.simpleOpenAIExecutor
import ai.koog.agents.ext.agent.reActStrategy
import ai.koog.agents.core.tools.ToolRegistry
import ai.koog.prompt.executor.clients.openai.OpenAIModels

val apiKey = System.getenv("OPENAI_API_KEY") ?: error("Please set OPENAI_API_KEY environment variable")
val promptExecutor = simpleOpenAIExecutor(apiKey)
val toolRegistry = ToolRegistry.EMPTY
val model =  OpenAIModels.Reasoning.O4Mini
-->
```kotlin hl_lines="5-10"
val reActAgent = AIAgent(
    executor = promptExecutor,
    toolRegistry = toolRegistry,
    llmModel = model,
    // reActStrategy をエージェント戦略として設定
    strategy = reActStrategy(
        // オプションのパラメータ値を設定
        reasoningInterval = 1,
        name = "react_agent"
    )
)
```
<!--- KNIT example-predefined-strategies-04.kt -->

### パラメータ

`reActStrategy`関数は以下のパラメータを受け取ります。

| パラメータ            | 型     | デフォルト | 説明                                                         |
|:--------------------|:-------|:---------|:-------------------------------------------------------------|
| `reasoningInterval` | Int    | 1        | 推論ステップの間隔を指定します。0より大きい必要があります。  |
| `name`              | String | `re_act` | 戦略の名前です。                                             |

### 使用例

ここに、ReAct戦略がシンプルなバンキングエージェントでどのように機能するかを示す例を示します。

#### 1. ユーザー入力

ユーザーは初期プロンプトを送信します。たとえば、これは「先月いくら使いましたか？」のような質問にすることができます。

#### 2. 推論

エージェントは、ユーザー入力と推論プロンプトを受け取り、初期推論を実行します。推論は次のようになります。

```text
I need to follow these steps:
1. Get all transactions from last month
2. Filter out deposits (positive amounts)
3. Calculate total spending
```

#### 3. 行動と実行、フェーズ1

エージェントが前のステップで定義したアクション項目に基づいて、前月のすべての取引を取得するためのツールを実行します。

この場合、実行するツールは`get_transactions`であり、前月のすべての取引を取得する要求に一致する`startDate`および`endDate`引数と共に使用されます。

```text
{tool: "get_transactions", args: {startDate: "2025-05-19", endDate: "2025-06-18"}}
```

ツールは次のような結果を返します。

```text
[
  {date: "2025-05-25", amount: -100.00, description: "Grocery Store"},
  {date: "2025-05-31", amount: +1000.00, description: "Salary Deposit"},
  {date: "2025-06-10", amount: -500.00, description: "Rent Payment"},
  {date: "2025-06-13", amount: -200.00, description: "Utilities"}
]
```

#### 4. 推論

ツールによって返された結果を使用して、エージェントは再度推論を実行し、そのフローにおける次のステップを決定します。

```text
I have the transactions. Now I need to:
1. Remove the salary deposit of +1000.00
2. Sum up the remaining transactions
```

#### 5. 行動と実行、フェーズ2

前の推論ステップに基づいて、エージェントはツール引数として提供された金額を合計する`calculate_sum`ツールを呼び出します。推論により、取引から正の金額を削除するというアクションポイントも導き出されたため、ツール引数として提供される金額は負の金額のみです。

```text
{tool: "calculate_sum", args: {amounts: [-100.00, -500.00, -200.00]}}
```

ツールは最終結果を返します。

```text
-800.00
```

#### 6. 最終応答

エージェントは、計算された合計を含む最終応答（アシスタントメッセージ）を返します。

```text
You spent $800.00 last month on groceries, rent, and utilities.
```

### ReAct戦略を使用するタイミング

ReAct戦略は、特に次の用途に役立ちます。

-   多段階の推論を必要とする複雑なタスク
-   最終的な回答を提供する前に、エージェントが情報を収集する必要があるシナリオ
-   小さなステップに分解することで効果が得られる問題
-   分析的思考とツールの使用の両方を必要とするタスク

### 例

ここに、事前定義されたReAct戦略（`reActStrategy`）とエージェントが使用する可能性のあるツールを実装するAIエージェントのコード例を示します。

<!--- INCLUDE
import ai.koog.agents.core.agent.AIAgent
import ai.koog.prompt.executor.llms.all.simpleOpenAIExecutor
import ai.koog.agents.ext.agent.reActStrategy
import ai.koog.agents.core.tools.ToolRegistry
import ai.koog.prompt.executor.clients.openai.OpenAIModels
import ai.koog.agents.ext.tool.AskUser
import ai.koog.agents.ext.tool.SayToUser

typealias Input = String

typealias getTransactions = AskUser
typealias calculateSum = SayToUser

val apiKey = System.getenv("OPENAI_API_KEY") ?: error("Please set OPENAI_API_KEY environment variable")
val promptExecutor = simpleOpenAIExecutor(apiKey)
val toolRegistry = ToolRegistry.EMPTY
val model =  OpenAIModels.Reasoning.O4Mini
-->
```kotlin
val bankingAgent = AIAgent(
    executor = promptExecutor,
    llmModel = model,
    // reActStrategy をエージェント戦略として使用
    strategy = reActStrategy(
        reasoningInterval = 1,
        name = "banking_agent"
    ),
    // エージェントが使用できるツールを追加
    toolRegistry = ToolRegistry {
        tool(getTransactions)
        tool(calculateSum)
    }
)

suspend fun main() { 
    // ユーザーのクエリでエージェントを実行
    val result = bankingAgent.run("How much did I spend last month?")
}
```
<!--- KNIT example-predefined-strategies-05.kt -->