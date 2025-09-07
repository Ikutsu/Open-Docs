[//]: # (title: Sevalla)

<show-structure for="chapter" depth="2"/>

<link-summary>了解如何準備並將 Ktor 應用程式部署到 Sevalla。</link-summary>

在本教學中，您將學習如何準備並將 Ktor 應用程式部署到 [Sevalla](https://sevalla.com/)。您可以根據用於[建立 Ktor 伺服器](server-create-and-configure.topic)的方式，使用以下其中一個初始專案：

* [embedded-server](https://github.com/ktorio/ktor-documentation/tree/%ktor_version%/codeSnippets/snippets/embedded-server)

* [Engine-main](https://github.com/ktorio/ktor-documentation/tree/%ktor_version%/codeSnippets/snippets/engine-main)

## 前提 {id="prerequisites"}

在開始本教學之前，您需要[建立一個 Sevalla 帳戶](https://sevalla.com)（附帶 $50 的免費額度）。

## 複製範例應用程式 {id="clone-sample-app"}

若要開啟範例應用程式，請按照以下步驟操作：

1. 複製 [Ktor 文件儲存庫](https://github.com/ktorio/ktor-documentation)。
2. 開啟 [codeSnippets](https://github.com/ktorio/ktor-documentation/tree/%ktor_version%/codeSnippets) 專案。
3. 開啟 [embedded-server](https://github.com/ktorio/ktor-documentation/tree/%ktor_version%/codeSnippets/snippets/embedded-server) 或 [engine-main](https://github.com/ktorio/ktor-documentation/tree/%ktor_version%/codeSnippets/snippets/engine-main) 範例，這兩個範例展示了設定 Ktor 伺服器的兩種不同方法 — 透過程式碼直接配置，或透過外部組態檔。部署這些專案的唯一區別在於如何指定用於監聽傳入請求的連接埠。

## 準備應用程式 {id="prepare-app"}

### 步驟 1: 配置連接埠 {id="port"}

Sevalla 會使用 `PORT` 環境變數注入一個隨機連接埠。您的應用程式必須配置為監聽該連接埠。

如果您選擇了在程式碼中指定伺服器配置的 [embedded-server](https://github.com/ktorio/ktor-documentation/tree/%ktor_version%/codeSnippets/snippets/embedded-server) 範例，您可以使用 `System.getenv()` 取得環境變數值。開啟位於 <Path>src/main/kotlin/com/example</Path> 資料夾中的 <Path>Application.kt</Path> 檔案，並如下所示更改 `embeddedServer()` 函數的連接埠參數值：

```kotlin
fun main() {
    val port = System.getenv("PORT")?.toIntOrNull() ?: 8080
    embeddedServer(Netty, port = port, host = "0.0.0.0") {
        // ...
    }.start(wait = true)
}
```

如果您選擇了在 <Path>application.conf</Path> 檔案中指定伺服器配置的 [engine-main](https://github.com/ktorio/ktor-documentation/tree/%ktor_version%/codeSnippets/snippets/engine-main) 範例，您可以使用 `${ENV}` 語法將環境變數分配給連接埠參數。開啟位於 <Path>src/main/resources</Path> 中的 <Path>application.conf</Path> 檔案，並如下所示更新它：

```hocon
ktor {
  deployment {
    port = 5000
    port = ${?PORT}
  }
  application {
    modules = [ com.example.ApplicationKt.module ]
  }
}
```

### 步驟 2: 新增 Dockerfile {id="dockerfile"}

若要在 Sevalla 上建構並執行您的 Ktor 專案，您需要一個 Dockerfile。這是一個使用多階段建構的 Dockerfile 範例：

```docker
# Stage 1: Build the app
FROM gradle:8.5-jdk17-alpine AS builder
WORKDIR /app
COPY . .
RUN gradle installDist

# Stage 2: Run the app
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/build/install/<project-name>/ ./
ENV PORT=8080
CMD ["./bin/<project-name>"]
```

請務必將 `<project-name>` 替換為您在 <Path>settings.gradle.kts</Path> 檔案中定義的專案名稱：

```kotlin
rootProject.name = "ktor-app"
```

## 部署應用程式 {id="deploy-app"}

Sevalla 會直接從連接的 Git 儲存庫建構和部署您的應用程式。這可以託管在 GitHub、GitLab、Bitbucket 或任何支援的 Git 提供者等平台上。若要成功部署，請確保您的專案已提交並推送，並包含所有必要檔案（例如您的 Dockerfile、<Path>build.gradle.kts</Path> 和原始碼）。

若要部署應用程式，請登入 [Sevalla](https://sevalla.com/) 並按照以下步驟操作：

1. 點擊 **應用程式 -> 建立應用程式**
  ![Sevalla 新增應用程式](../images/sevalla-add-app.jpg)
2. 選擇您的 Git 儲存庫並選取適當的分支（通常是 `main` 或 `master`）。
3. 設定 **應用程式名稱**，選取 **區域**，並選擇您的 **pod 大小**（您可以從 0.5 CPU / 1GB RAM 開始）。
4. 點擊 **建立**，但目前請跳過部署步驟
  ![Sevalla 建立應用程式](../images/sevalla-deployment-create-app.png)
5. 前往 **設定 -> 建構**，然後點擊 **建構環境** 卡片下的 **更新設定**。
  ![Sevalla 更新建構設定](../images/sevalla-deployment-update-build-settings.png)
6. 將建構方法設定為 **Dockerfile**。
  ![Sevalla Dockerfile 設定](../images/sevalla-deployment-docker-settings.png)
7. 確認 **Dockerfile 路徑** 是 `Dockerfile` 且 **上下文** 是 `.`。
8. 返回您應用程式的 **部署** 標籤並點擊 **部署**。

Sevalla 將複製您的 Git 儲存庫，使用您的 Dockerfile 建構 Docker 映像，注入 `PORT` 環境變數，並執行您的應用程式。如果一切配置正確，您的 Ktor 應用程式將在 `https://<your-app>.sevalla.app` 上線。