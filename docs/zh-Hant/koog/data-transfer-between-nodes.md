## 概述

Koog 提供了一種使用 `AIAgentStorage` 儲存和傳遞資料的方式，`AIAgentStorage` 是一個鍵值儲存系統，設計用於在不同節點甚至子圖之間型別安全地傳遞資料。

儲存機制可透過代理節點中可用的 `storage` 屬性（`storage: AIAgentStorage`）存取，從而實現 AI 代理系統不同元件之間的無縫資料共享。

## 鍵與值結構

鍵值資料儲存結構依賴於 `AIAgentStorageKey` 資料類別。有關 `AIAgentStorageKey` 的更多資訊，請參閱以下章節。

### AIAgentStorageKey

儲存機制使用型別化鍵系統，以確保儲存和檢索資料時的型別安全：

- `AIAgentStorageKey<T>`：一個資料類別，代表用於識別和存取資料的儲存鍵。`AIAgentStorageKey` 類別的主要特性如下：
    - 泛型型別參數 `T` 指定了與此鍵相關聯的資料型別，確保了型別安全。
    - 每個鍵都有一個 `name` 屬性，它是一個字串識別符，唯一地代表儲存鍵。

## 使用範例

以下章節提供了一個實際範例，說明如何建立儲存鍵並使用它來儲存和檢索資料。

### 定義一個代表您資料的類別

儲存您想要傳遞的資料的第一步是建立一個代表您資料的類別。這是一個包含基本使用者資料的簡單類別範例：

```kotlin
class UserData(
   val name: String,
   val age: Int
)
```
<!--- KNIT example-data-transfer-between-nodes-01.kt -->

定義後，請按如下所述使用該類別建立儲存鍵。

### 建立儲存鍵

為已定義的資料結構建立一個型別化儲存鍵：

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

`createStorageKey` 函數接受一個單一字串參數，該參數唯一識別該鍵。

### 儲存資料

若要使用已建立的儲存鍵儲存資料，請在節點中使用 `storage.set(key: AIAgentStorageKey<T>, value: T)` 方法：

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

### 檢索資料

若要檢索資料，請在節點中使用 `storage.get` 方法：

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

## API 文件

有關 `AIAgentStorage` 類別的完整參考，請參閱 [AIAgentStorage](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/index.html)。

有關 `AIAgentStorage` 類別中可用的各別函數，請參閱以下 API 參考資料：

- [clear](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/clear.html)
- [get](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/get.html)
- [getValue](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/get-value.html)
- [putAll](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/put-all.html)
- [remove](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/remove.html)
- [set](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/set.html)
- [toMap](https://api.koog.ai/agents/agents-core/ai.koog.agents.core.agent.entity/-a-i-agent-storage/to-map.html)

## 額外資訊

- `AIAgentStorage` 是執行緒安全的，使用互斥鎖 (Mutex) 以確保並發存取得到正確處理。
- 此儲存設計為可與任何繼承 `Any` 的型別協同工作。
- 檢索值時，型別轉換會自動處理，確保整個應用程式的型別安全。
- 若要進行非空值存取，請使用 `getValue` 方法，如果鍵不存在，該方法會拋出一個例外。
- 您可以使用 `clear` 方法完全清除儲存，該方法會移除所有已儲存的鍵值對。