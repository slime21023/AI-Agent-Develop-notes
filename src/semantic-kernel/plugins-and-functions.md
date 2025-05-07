# 插件與函數 (Plugins and Functions)

在我們探索 Semantic Kernel 這個強大框架時，**插件 (Plugins)** 與 **函數 (Functions)** 是不可或缺的關鍵角色。它們是我們賦予 AI 應用程式具體「技能」和「知識」的核心構件，讓 AI 不僅能「思考」，更能「行動」。

## 核心構建塊：函數 (Function)

想像一下，您正在組裝一個機器人。**函數 (Function)** 就是這個機器人的每一個獨立工具或能力，是 Semantic Kernel 中最小的、可執行的「技能單位」。

Semantic Kernel 的函數非常靈活，主要有以下幾種類型：

1.  **原生程式碼 (Native Code)**：
    *   **組成**: 用您熟悉的程式語言（如 Python、C#）編寫的程式碼。
    *   **用途**: 執行應用程式的特定邏輯，比如數學計算、檔案操作、資料庫互動，或任何您希望 AI 能夠執行的客製化任務。
    *   **例子**: 一個能計算商品折扣的 Python 函數，或是一個能從公司內部系統查詢訂單狀態的 C# 函數。

2.  **提示詞模板 (Prompt Templates)**：
    *   **組成**: 預先設計好的「指令稿」，用來引導大型語言模型 (LLM) 如何生成回應或執行任務。通常包含可動態填入的變數。
    *   **用途**: 這是與 AI 模型溝通和賦予其創造性任務（如寫作、摘要、翻譯）的主要方式。
    *   **例子**: 一個用於生成產品描述的模板，其中包含 `{{產品名稱}}` 和 `{{特色}}` 等變數，AI 會根據這些變數生成吸引人的描述。

3.  **OpenAPI 規格 (OpenAPI Specs)**：
    *   **組成**: 如果您的 AI 需要與外部的網路服務 (Web API) 互動，例如獲取即時天氣、查詢股票價格或連接到您公司其他的 API 服務，您可以使用 OpenAPI 規格 (也稱為 Swagger) 來定義這些服務的介面。
    *   **用途**: 讓 Semantic Kernel 能夠理解並安全地調用這些外部 API，從而極大地擴展 AI 的能力範圍。
    *   **例子**: 一個定義了如何透過 API 查詢特定城市目前天氣狀況的 OpenAPI 文件。

4.  **ITextSearch 實作 (用於 RAG 場景)**：
    *   **組成**: 一種特殊的函數，專門用於在大量文本資料中進行高效搜尋。
    *   **用途**: 主要應用於「檢索增強生成」(Retrieval Augmented Generation, RAG) 的場景。在 RAG 中，AI 會先從一個或多個知識庫 (如公司內部文件、產品手冊、FAQ) 中搜尋相關資訊，然後利用這些檢索到的資訊來生成更準確、更具上下文的回答。
    *   **例子**: 一個能夠在您的技術支援知識庫中搜尋相關文章，以回答用戶問題的函數。

## 技能的容器：插件 (Plugin)

如果說函數是單個的工具，那麼 **插件 (Plugin)** 就是一個**工具箱**。它是一個包含一個或多個相關函數的容器。

例如，您可以創建一個名為「文件處理 (DocumentTools)」的插件，裡面可能包含：

*   `SummarizeText` (文本摘要函數，可能是一個提示詞模板)
*   `TranslateText` (文本翻譯函數，可能也是一個提示詞模板)
*   `ExtractKeywords` (提取關鍵詞函數，可能是原生程式碼或提示詞模板)

**使用插件的好處：**

*   **組織性：** 將相關功能的函數組織在一起，使您的 AI 專案結構更清晰、更易於維護。
*   **可重用性：** 精心設計的插件可以在不同的 AI 應用或專案中重複使用。
*   **模組化：** 讓您可以像搭積木一樣，靈活組合不同的插件來建構功能豐富的 AI 應用。

## 如何創建和使用插件？

要讓 Kernel (Semantic Kernel 的核心協調器) 和 AI 模型能夠使用您定義的函數，您需要先將這些函數組織到插件中，然後將插件「註冊」到 Kernel。一旦註冊，AI (通常是透過聊天完成服務或規劃器) 就可以根據用戶的請求或應用程式的流程，智能地發現並調用這些函數。

讓我們看看一個簡單的 Python 程式碼範例，了解如何定義一個插件和其中的函數：

```python
# 假設 kernel 變數已經被初始化並配置了 AI 服務
# import semantic_kernel as sk
# kernel = sk.Kernel()
# ... (省略 kernel 初始化和服務添加的程式碼)

from semantic_kernel.functions import kernel_function # 匯入用於標記函數的裝飾器

# 1. 定義一個插件類別 (Plugin Class)
#    類別的名稱通常能反映插件的功能，例如 LightsPlugin, EmailPlugin 等。
class LightsPlugin:
    """一個用於控制燈光開關的插件。"""

    # 2. 在插件類別中定義一個或多個函數 (Function)
    #    使用 @kernel_function 裝飾器來告訴 Semantic Kernel 這是一個可被 AI 調用的函數。
    @kernel_function(
        description="根據指令開啟或關閉燈光 (Turn the lights on or off)" # 函數的描述至關重要！AI 會參考它來理解函數的功能和適用場景。
        # name="control_lights" # 您可以選擇性地指定函數名稱，如果未指定，則默認為 Python 方法的名稱。
    )
    def control_lights(self, state: str) -> str: # 函數可以接受參數，並應有明確的返回類型。
        """
        改變燈光的狀態。
        :param state: 期望的燈光狀態，例如 "on" 或 "off"。
        :return: 一個表示操作結果的字串。
        """
        print(f"LightsPlugin: 收到指令，將燈光狀態設置為 {state}")
        return f"燈光已成功設置為 {state}。"

# 3. 創建插件的實例 (Instance)
my_lights_plugin = LightsPlugin()

# 4. 將插件實例添加到 Kernel 中
#    plugin_name 參數為您的插件指定一個在 Kernel 中唯一的名稱。
#    AI 在需要調用此插件中的函數時，會使用這個名稱。
kernel.add_plugin(my_lights_plugin, plugin_name="Lights")
```

### 總結

插件和函數是 Semantic Kernel 的核心組成部分，它們讓您能夠將複雜的任務分解為更小、可管理的技能單元，並將這些技能賦予您的 AI 應用。

透過精心設計的函數描述和合理的插件組織，您可以創建出功能強大、能夠與外部世界互動並執行實際任務的智能助手。