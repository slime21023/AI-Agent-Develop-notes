# AI 服務連接器 (AI Service Connectors)

AI 服務連接器（AI Service Connectors）是 Microsoft 開源框架 **Semantic Kernel** 的核心組件之一，它透過抽象化技術，將不同供應商的 AI 服務（如 OpenAI、Hugging Face 等）整合到單一介面中。這種設計讓開發者能專注於業務邏輯，無需重寫程式碼即可切換 AI 模型供應商


## 為什麼需要連接器？
傳統 AI 整合常面臨以下痛點：
1. **供應商綁定**：更換模型需大幅修改程式
2. **介面不一致**：不同服務的 API 格式各異
3. **功能碎片化**：需自行處理錯誤重試、日誌記錄等基礎功能  

AI 服務連接器透過標準化介面解決了這些問題，例如：
```python
# 切換至 Hugging Face 僅需替換類別
from semantic_kernel.connectors.ai.hugging_face import HuggingFaceTextGeneration

kernel.add_service(HuggingFaceTextGeneration(
    model_id="bigscience/bloom",
    api_key="your-huggingface-key"
))
```
此特性讓開發者能輕鬆比較不同模型的效能與成本。


## 支援的 AI 服務類型
| 服務類型         | 應用場景                          | 常用模型範例              |
|------------------|-----------------------------------|--------------------------|
| 聊天完成         | 對話式互動（如客服機器人）        | GPT-4、Claude 2          |
| 文字生成         | 文章撰寫、程式碼生成              | GPT-3.5、Llama 2         |
| 嵌入生成         | 語義搜尋、向量資料庫應用          | text-embedding-ada-002   |
| 文字轉圖片       | 圖像創作、設計輔助                | DALL-E 3、Stable Diffusion|
| 多模態轉換       | 語音助手、媒體內容分析            | Whisper、TTS-1           |


## 範例

### 註冊服務
```python
from semantic_kernel import Kernel
from semantic_kernel.connectors.ai.open_ai import OpenAIChatCompletion

kernel = Kernel()
kernel.add_service(OpenAIChatCompletion(
    model_id="gpt-4-turbo", 
    api_key="sk-xxx"
))
```

### 調用服務
```python
prompt = "用繁體中文解釋量子計算"
response = await kernel.invoke(
    kernel.create_function_from_prompt(prompt),
    temperature=0.7
)
print(response)
```


## 進階應用技巧
**混合使用多種服務**: 可同時註冊多個服務，根據需求動態選擇
```python
# 註冊 OpenAI 與 Azure AI 端點
kernel.add_service(OpenAIChatCompletion(...))
kernel.add_service(AzureChatCompletion(...))

# 根據成本或效能需求切換服務
if is_cost_sensitive:
    service = kernel.get_service("azure_chat")
else:
    service = kernel.get_service("openai_chat")
```

**自定義 Connector**: 透過繼承 `TextGenerationClientBase` 類別，可擴充支援私有化部署的模型
```python
class CustomAIClient(TextGenerationClientBase):
    async def generate_text(self, prompt: str) -> str:
        # 實作自定義調用邏輯
        return await call_custom_api(prompt)
```

## 常見問題
❓ **如何選擇預設服務？**  
系統會優先使用最後註冊的服務，建議透過 `kernel.get_service()` 明確指定。

❓ **支援本地模型嗎？**  
是的，需透過自定義 Connector 整合 LM Studio、Ollama 等本地推理框架。

❓ **錯誤處理機制為何？**  
內建自動重試、速率限制與 Fallback 機制。
