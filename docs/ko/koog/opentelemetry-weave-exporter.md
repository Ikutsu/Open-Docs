# W&B Weave 익스포터

Koog는 AI 애플리케이션의 가시성과 분석을 위한 Weights & Biases의 개발자 도구인 [W&B Weave](https://wandb.ai/site/weave/)로 에이전트 트레이스를 내보내는 내장 지원을 제공합니다. Weave 통합을 통해 프롬프트, 완성본, 시스템 컨텍스트 및 실행 트레이스를 캡처하고 W&B 워크스페이스에서 직접 시각화할 수 있습니다.

Koog의 OpenTelemetry 지원에 대한 배경 정보는 [OpenTelemetry 지원](https://docs.koog.ai/opentelemetry-support/) 문서를 참조하십시오.

---

## 설정 지침

1.  [https://wandb.ai](https://wandb.ai)에서 W&B 계정을 생성합니다.
2.  [https://wandb.ai/authorize](https://wandb.ai/authorize)에서 API 키를 가져옵니다.
3.  [https://wandb.ai/home](https://wandb.ai/home)에서 W&B 대시보드를 방문하여 엔티티 이름을 찾습니다. 엔티티는 개인 계정인 경우 일반적으로 사용자 이름이며, 팀/조직 계정인 경우 팀/조직 이름입니다.
4.  프로젝트 이름을 정의합니다. 프로젝트를 미리 생성할 필요는 없으며, 첫 번째 트레이스가 전송될 때 자동으로 생성됩니다.
5.  Weave 엔티티, 프로젝트 이름 및 API 키를 Weave 익스포터에 전달합니다. 이는 `addWeaveExporter()` 함수에 매개변수로 제공하거나 아래와 같이 환경 변수를 설정하여 수행할 수 있습니다:

```bash
export WEAVE_API_KEY="<your-api-key>"
export WEAVE_ENTITY="<your-entity>"
export WEAVE_PROJECT_NAME="koog-tracing"
```

## 구성

Weave 익스포트를 활성화하려면 **OpenTelemetry 기능**을 설치하고 `WeaveExporter`를 추가합니다. 익스포터는 `OtlpHttpSpanExporter`를 통해 Weave의 OpenTelemetry 엔드포인트를 사용합니다.

### 예시: Weave 트레이싱을 사용하는 에이전트

<!--- INCLUDE
import ai.koog.agents.core.agent.AIAgent
import ai.koog.agents.features.opentelemetry.feature.OpenTelemetry
import ai.koog.agents.features.opentelemetry.integration.weave.addWeaveExporter
import ai.koog.prompt.executor.clients.openai.OpenAIModels
import ai.koog.prompt.executor.llms.all.simpleOpenAIExecutor
import kotlinx.coroutines.runBlocking
-->
```kotlin
fun main() = runBlocking {
    val apiKey = "api-key"
    val entity = System.getenv()["WEAVE_ENTITY"] ?: throw IllegalArgumentException("WEAVE_ENTITY is not set")
    val projectName = System.getenv()["WEAVE_PROJECT_NAME"] ?: "koog-tracing"
    
    val agent = AIAgent(
        executor = simpleOpenAIExecutor(apiKey),
        llmModel = OpenAIModels.CostOptimized.GPT4oMini,
        systemPrompt = "You are a code assistant. Provide concise code examples."
    ) {
        install(OpenTelemetry) {
            addWeaveExporter()
        }
    }

    println("Running agent with Weave tracing")

    val result = agent.run("Tell me a joke about programming")

    println("Result: $result
See traces on https://wandb.ai/$entity/$projectName/weave/traces")
}
```
<!--- KNIT example-weave-exporter-01.kt -->

## 트레이스되는 내용

활성화되면 Weave 익스포터는 Koog의 일반 OpenTelemetry 통합과 마찬가지로 다음을 포함한 동일한 스팬을 캡처합니다.

-   **에이전트 라이프사이클 이벤트**: 에이전트 시작, 중지, 오류
-   **LLM 상호작용**: 프롬프트, 완성본, 지연 시간
-   **도구 호출**: 도구 호출을 위한 실행 트레이스
-   **시스템 컨텍스트**: 모델 이름, 환경, Koog 버전과 같은 메타데이터

W&B Weave에서 시각화될 때 트레이스는 다음과 같이 나타납니다:
![W&B Weave 트레이스](img/opentelemetry-weave-exporter-light.png#only-light)
![W&B Weave 트레이스](img/opentelemetry-weave-exporter-dark.png#only-dark)

더 자세한 내용은 공식 [Weave OpenTelemetry 문서](https://weave-docs.wandb.ai/guides/tracking/otel/)를 참조하십시오.

---

## 문제 해결

### Weave에 트레이스가 나타나지 않음
-   `WEAVE_API_KEY`, `WEAVE_ENTITY`, `WEAVE_PROJECT_NAME`가 환경에 설정되어 있는지 확인하십시오.
-   W&B 계정이 지정된 엔티티 및 프로젝트에 대한 접근 권한을 가지고 있는지 확인하십시오.

### 인증 오류
-   `WEAVE_API_KEY`가 유효한지 확인하십시오.
-   API 키는 선택된 엔티티에 대한 트레이스를 작성할 권한이 있어야 합니다.

### 연결 문제
-   환경에 W&B의 OpenTelemetry 수집 엔드포인트에 대한 네트워크 접근 권한이 있는지 확인하십시오.