# Kernel

Kernel 是 Semantic Kernel 的核心組件，就像 AI 應用的大腦。它是一個**依賴注入容器**，專門負責管理所有運行 AI 應用所需的服務和插件。無論你需要語言模型、資料查詢還是其他外部功能，Kernel 都能協助你統一調度與管理。

### Kernel 的主要功能

1. **處理提示（Prompt）並選擇最佳 AI 服務**  
   當你向 AI 應用發出請求時，Kernel 會自動處理這些提示，並根據需求選擇最合適的 AI 服務提供回應。

2. **構建並發送提示到 AI 服務**  
   Kernel 可以協助你將請求內容（Prompt）包裝成適合 AI 服務的格式，並發送給對應的服務。

3. **解析並返回 AI 回應**  
   從 AI 服務獲得回應後，Kernel 會解析這些資料，再傳回給你的應用程式，讓你能輕鬆取得需要的結果。

4. **支援事件和中間件觸發**  
   Kernel 支援事件與中間件機制，開發者可以利用這些功能進行日誌記錄、狀態追蹤、錯誤處理等進階操作，讓應用更靈活可靠。

## 如何配置 Kernel？

對初學者來說，Kernel 的配置非常直覺。你只需要透過單一入口點，就可以將不同的 AI 服務和插件加入 Kernel，實現統一管理。以下是一個簡單的 Python 範例，說明如何配置 Kernel 並加入 Azure OpenAI 服務：

```python
import semantic_kernel as sk
from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion

# 建立 Kernel 實例
kernel = sk.Kernel()

# 設定 Azure OpenAI 聊天補全服務
# 請替換為你自己的部署名稱、端點與 API 金鑰
deployment_name = "your-deployment-name"
endpoint = "your-endpoint"
api_key = "your-api-key"

# 加入 AI 服務到 Kernel
kernel.add_service(AzureChatCompletion(
    deployment_name=deployment_name,
    endpoint=endpoint,
    api_key=api_key
))
```

這樣設定後，Kernel 就會自動管理這些服務，當你需要使用時可以直接調用，無需重複配置。