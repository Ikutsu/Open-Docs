## 概要

Koogは、`AIAgentStorage`を使用してデータを保存および受け渡す方法を提供します。これは、異なるノード間やサブグラフ間でもデータを型安全な方法で受け渡すように設計されたキーバリュー型ストレージシステムです。

このストレージは、エージェントノード内で利用可能な`storage`プロパティ（`storage: AIAgentStorage`）を通じてアクセスでき、AIエージェントシステムの様々なコンポーネント間でシームレスなデータ共有を可能にします。

## キーと値の構造

キーバリュー型データストレージの構造は、`AIAgentStorageKey`データクラスに依存しています。`AIAgentStorageKey`の詳細については、以下のセクションを参照してください。

### AIAgentStorageKey

ストレージは、データの保存と取得の際に型の安全性を確保するために、型付きキーシステムを使用します。

- `AIAgentStorageKey<T>`: データを識別しアクセスするために使用されるストレージキーを表すデータクラスです。`AIAgentStorageKey`クラスの主な特徴は以下の通りです。
    - ジェネリック型パラメータ`T`は、このキーに関連付けられるデータの型を指定し、型の安全性を確保します。
    - 各キーには、ストレージキーを一意に識別する文字列識別子である`name`プロパティがあります。

## 使用例

以下のセクションでは、ストレージキーを作成し、それを使用してデータを保存および取得する実際の例を提供します。

### データを表すクラスの定義

渡したいデータを保存する最初のステップは、そのデータを表現するクラスを作成することです。以下に、基本的なユーザーデータを持つシンプルなクラスの例を示します。

```kotlin
class UserData(
   val name: String,
   val age: Int
)
```
<!--- KNIT example-data-transfer-between-nodes-01.kt -->

一度定義したら、以下で説明するように、そのクラスを使用してストレージキーを作成します。

### ストレージキーの作成

定義したデータ構造に対して型付きストレージキーを作成します。

<!--- INCLUDE
import ai.koog.agents.core.agent.entity.createStorageKey

class UserData(
    val name: String,
    val age: Int
)

fun main() {
-->
<!--- SUFFIX
}
-->
```kotlin
val userDataKey = createStorageKey<UserData>("user-data")
```
<!--- KNIT example-data-transfer-between-nodes-02.kt -->

`createStorageKey`関数は、キーを一意に識別する単一の文字列パラメータを取ります。

### データの保存

作成したストレージキーを使用してデータを保存するには、ノード内で`storage.set(key: AIAgentStorageKey<T>, value: T)`メソッドを使用します。

<!--- INCLUDE
import ai.koog.agents.core.dsl.builder.strategy
import ai.koog.agents.core.agent.entity.createStorageKey

class UserData(
   val name: String,
   val age: Int
)

fun main() {
    val userDataKey = createStorageKey<UserData>("user-data")

    val str = strategy<Unit, Unit>("my-strategy") {
-->
<!--- SUFFIX
    }
}
-->
```kotlin
val nodeSaveData by node<Unit, Unit> {
    storage.set(userDataKey, UserData("John", 26))
}
```
<!--- KNIT example-data-transfer-between-nodes-03.kt -->

### データの取得

データを取得するには、ノード内で`storage.get`メソッドを使用します。

<!--- INCLUDE
import ai.koog.agents.core.agent.entity.createStorageKey
import ai.koog.agents.core.dsl.builder.strategy

class UserData(
    val name: String,
    val age: Int
)

fun main() {
    val userDataKey = createStorageKey<UserData>("user-data")

    val str = strategy<String, Unit>("my-strategy") {
-->
<!--- SUFFIX
    }
}
-->
```kotlin
val nodeRetrieveData by node<String, Unit> { message ->
    storage.get(userDataKey)?.let { userFromStorage ->
        println("Hello dear $userFromStorage, here's a message for you: $message")
    }
}
```
<!--- KNIT example-data-transfer-between-nodes-04.kt -->

## APIドキュメント

`AIAgentStorage`クラスに関する完全なリファレンスについては、[AIAgentStorage](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/index.html)を参照してください。

`AIAgentStorage`クラスで利用可能な個々の関数については、以下のAPIリファレンスを参照してください。

- [clear](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/clear.html)
- [get](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/get.html)
- [getValue](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/get-value.html)
- [putAll](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/put-all.html)
- [remove](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/remove.html)
- [set](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/set.html)
- [toMap](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/to-map.html)

## 追加情報

- `AIAgentStorage`はスレッドセーフであり、Mutexを使用して同時アクセスが適切に処理されることを保証します。
- このストレージは、`Any`を継承する任意の型で動作するように設計されています。
- 値を取得する際、型変換は自動的に処理され、アプリケーション全体で型の安全性が確保されます。
- 値への非Null許容（non-nullable）アクセスには、キーが存在しない場合に例外をスローする`getValue`メソッドを使用します。
- `clear`メソッドを使用すると、ストレージ全体をクリアでき、保存されているすべてのキーと値のペアが削除されます。