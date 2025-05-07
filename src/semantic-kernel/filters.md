# 過濾器 (Filters)

在使用 Semantic Kernel 建構 AI 應用時，我們經常需要在核心功能（如函數調用或提示詞渲染）的執行流程中插入一些自訂的邏輯。無論是為了監控、日誌記錄、修改輸入輸出，還是實現更複雜的控制策略，**過濾器 (Filters)** 都提供了一個優雅且強大的解決方案。

過濾器允許您在 Kernel 的關鍵操作點「掛鉤」(hook)，在這些操作執行之前或之後執行您的程式碼，而無需修改 Kernel 或插件的內部實現。這種類似於「面向切面程式設計 (Aspect-Oriented Programming, AOP)」的機制，能讓您的程式碼更加模組化和易於維護。

## 什麼是過濾器 (Filters)？

在 Semantic Kernel 中，過濾器是一種可攔截函數調用和提示詞渲染過程的機制。您可以把它想像成一系列的「檢查點」或「處理站」，當 Kernel 執行特定操作時，會依序通過這些已註冊的過濾器。

**主要用途：**

1.  **監控與日誌 (Monitoring and Logging):** 記錄函數調用的頻率、耗時、輸入參數、輸出結果等，方便追蹤和分析。
2.  **輸入/輸出修改 (Input/Output Modification):** 在函數執行前修改傳入的參數，或在函數執行後修改其返回結果。例如，自動格式化輸入，或對輸出進行脫敏處理。
3.  **驗證與校驗 (Validation):** 在執行前檢查參數是否符合預期，或在執行後驗證結果的有效性。
4.  **快取 (Caching):** 對於相同的輸入，如果之前已經計算過結果，可以直接從快取返回，避免重複執行耗時操作。
5.  **錯誤處理與重試 (Error Handling and Retry Logic):** 捕獲特定異常，並執行自訂的錯誤處理邏輯，例如重試。
6.  **成本與用量追蹤 (Cost and Token Tracking):** 估算或記錄與 LLM 互動相關的成本或 token 使用量。
7.  **策略執行 (Policy Enforcement):** 例如，實現速率限制、內容審查等。

## 過濾器的核心：`Filter` 介面與生命週期方法

要創建一個自訂過濾器，您需要繼承 Semantic Kernel 提供的 `Filter` 基底類別 (或其特定子類，如 `FunctionInvocationFilterBase`, `PromptRenderFilterBase`，視具體版本和需求而定)，並覆寫其提供的一個或多個「生命週期方法」。

這些方法會在 Kernel 執行流程的特定階段被調用：

*   **針對函數調用 (Function Invocation):**
    *   `on_function_invoking(context: FunctionInvokingContext)`: 在函數即將被調用之前執行。您可以在這裡讀取或修改傳入的參數。如果在此方法中將 `context.cancel` 設為 `True`，則可以阻止後續的函數調用。
    *   `on_function_invoked(context: FunctionInvokedContext)`: 在函數調用完成之後執行。您可以在這裡讀取函數的執行結果，或修改最終返回給調用者的結果。

*   **針對提示詞渲染 (Prompt Rendering):**
    *   `on_prompt_rendering(context: PromptRenderingContext)`: 在提示詞模板即將被渲染 (即變數被實際值替換) 之前執行。
    *   `on_prompt_rendered(context: PromptRenderedContext)`: 在提示詞模板渲染完成之後執行，此時您可以獲取到最終將發送給 AI 服務的完整提示詞。

每個生命週期方法都會接收一個 `context` 物件作為參數。這個 `context` 物件非常重要，它攜帶了當前執行點的相關資訊，例如：

*   對於函數調用：Kernel 實例、被調用的函數 (KernelFunction)、傳入的參數 (KernelArguments)、函數執行結果 (FunctionResult) 等。
*   對於提示詞渲染：Kernel 實例、原始提示詞模板、渲染後的提示詞等。

## 實作範例：一個簡單的日誌過濾器

```python
from semantic_kernel.filters import Filter
from semantic_kernel.filters.functions import FunctionInvokedContext
# from semantic_kernel.kernel import Kernel # 若要實際運行 kernel.add_filter，需導入 Kernel

class LoggingFilter(Filter):
    async def on_function_invoked(self, context: FunctionInvokedContext) -> None:
        """
        當一個函數被調用完成後，此方法會被執行。
        它會獲取函數的名稱並打印一條日誌訊息。
        """
        # 從 context.function (KernelFunctionMetadata) 獲取函數的元數據
        func_metadata = context.function
        
        # 構建函數的顯示名稱 (例如：PluginName.FunctionName 或 FunctionName)
        # 這是比直接使用 'context.function_name' (若存在) 更標準且可靠的方式。
        function_display_name = func_metadata.name
        if func_metadata.plugin_name:
            function_display_name = f"{func_metadata.plugin_name}.{function_display_name}"
            
        # 打印日誌，格式與您最初提供的範例相似
        print(f"Function {function_display_name} invoked")

# 假設 'kernel' 是一個已經初始化好的 Semantic Kernel 實例
# kernel = Kernel() 

# 1. 創建過濾器的實例
my_logging_filter = LoggingFilter()

# 2. 將過濾器實例添加到 Kernel
#    您最初的範例使用了 kernel.add_filter(LoggingFilter())。
#    這種方式在某些 SK 版本中可能直接有效，或者需要更明確地指定過濾器類型。
#    例如: kernel.add_filter("function_invocation", my_logging_filter)
#    為保持簡潔並貼近您的原始範例風格，我們假設以下方式適用：
kernel.add_filter(my_logging_filter)
```

### 四、過濾器的威力與使用場景

過濾器提供了一個非常靈活的方式來擴展和控制 Semantic Kernel 的行為：

*   **精細化日誌與追蹤**：不僅記錄函數名，還可以記錄參數、結果、執行時間、token 消耗等，用於調試和性能分析。
*   **輸入驗證與轉換**：在函數執行前，檢查輸入參數是否合法，或將其轉換為函數期望的格式。
*   **輸出處理**：對函數的原始輸出進行後處理，例如格式化、數據清洗、添加額外資訊，或根據特定條件修改結果。
*   **錯誤攔截與處理**：集中處理特定類型的異常，實現自訂的重試邏輯或回退策略。
*   **安全與合規**：實現 PII (個人身份資訊) 檢測與脫敏，或檢查請求/回應是否符合內容安全策略。
*   **動態配置修改**：在運行時根據某些條件修改傳遞給 AI 模型的請求參數（例如，動態調整 `temperature` 或 `max_tokens`）。