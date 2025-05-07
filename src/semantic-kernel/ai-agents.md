# 建構AI 代理 (Building AI Agents)

想像一下，您不僅能與 AI 對話，AI 還能主動為您執行任務、協調工作，甚至與其他 AI 協同合作解決複雜問題。這就是 **AI 代理 (AI Agents)** 的魅力所在。AI 代理可以被視為一個能夠感知其環境（例如，接收使用者輸入）、進行決策（例如，選擇下一步行動或調用哪個工具）並採取行動（例如，呼叫 API、生成回應或控制設備）的智能實體。Semantic Kernel 不僅能讓您建立強大的 AI 應用，更是建構這些智能代理的得力助手。

## 從簡單開始：您的第一個 AI 互動夥伴 (基本聊天機器人)

許多 AI 代理的核心始於一個能夠理解和回應使用者指令的互動介面，通常以聊天機器人的形式出現。Semantic Kernel 讓建構這樣的基礎聊天機器人變得直觀且高效。

**核心元件：**

1.  **聊天完成服務 (Chat Completion Service)：** 這是 AI 的「大腦」，通常是一個大型語言模型 (LLM)，如 OpenAI 的 GPT 系列或 Azure OpenAI Service 提供的模型。它負責理解輸入並生成自然語言回應，或者決定下一步的行動。
2.  **插件 (Plugins)：** 這些是賦予 AI 特定「技能」或「工具」的模組化程式碼。例如，一個插件可能包含開燈、查詢天氣、讀取日曆或與特定 API 互動的功能。插件使得 AI 不僅僅能聊天，還能執行實際操作，這是代理「行動」能力的關鍵。
3.  **聊天歷史 (Chat History)：** 為了讓對話能夠連貫，AI 需要記住之前的交流內容。聊天歷史就是用來儲存和管理對話上下文的。

**實作範例解析：**

```python
# 假設 'kernel' 是一個已經初始化並配置好 Semantic Kernel 實例，
# 並且已經註冊了聊天完成服務。
# 同時，假設有一個插件（例如 HomeAutomationPlugin）包含了一個名為 "control_lights" 的函數，
# 並且這個插件已經被添加到了 kernel 中。

from semantic_kernel.contents import ChatHistory # 引入聊天歷史記錄工具

# 1. 初始化聊天歷史
#    ChatHistory 用於追蹤對話的來回，讓 AI 能夠記住上下文。
history = ChatHistory()

# 2. 使用者發出指令
#    模擬使用者輸入一條訊息。
user_message = "Turn on the lights"
history.add_user_message(user_message)
# print(f"使用者: {user_message}")

# 3. Kernel 理解指令並調用相應的插件函數來執行動作
#    kernel.invoke_async 是 Semantic Kernel 中執行函數的核心方法。
#    - function_name="control_lights": 指定要運行的函數名稱。
#      這個函數通常定義在一個插件中，賦予 AI 特定的能力。
#      例如，一個 "HomeAutomation" 插件可能會有 control_lights, set_thermostat 等函數。
#      Kernel 會在其註冊的插件中尋找這個函數。
#    - arguments={"state": "on"}: 傳遞給 "control_lights" 函數的參數。
#      這裡告訴函數要將燈的狀態設置為 "on"。
#    result 將包含函數執行的結果。
result = await kernel.invoke_async(
    function_name="control_lights", # 或者 plugin_name="MyLightPlugin", function_name="control_lights"
    arguments={"state": "on"}
)

# 4. 將 AI (助手) 的回應或行動結果添加到歷史記錄中
#    result.value 通常包含了被調用函數返回的主要內容，例如一個確認訊息。
#    將此訊息作為 AI 的回應添加到聊天歷史中。
history.add_assistant_message(str(result.value)) # 確保是字串
```

> 它已經具備了感知（接收用戶訊息）、決策（Kernel 解析並選擇函數）和行動（調用插件函數）的初步能力。

## 邁向進階：複雜多代理系統 (Complex Multi-Agent Systems)

當任務變得更加複雜，或者需要不同領域的專業知識協同工作時，單一 AI 代理可能就顯得力不從心了。這時，**多代理系統 (Multi-Agent Systems)** 的概念就派上了用場。在這樣的系統中，多個具有不同專長、角色或目標的 AI 代理可以互相溝通、協調並共同完成更宏大的任務。

Semantic Kernel 提供了**代理框架 (Agent Framework)**，旨在支持和簡化這類複雜系統的建構。

**代理框架的目標與能力：**

1.  **多代理協作 (Multi-agent collaboration):**
    *   **單一對話中的協作：** 想像一個場景，使用者提出一個複雜請求，例如「幫我規劃一次為期一周的夏威夷家庭假期，需要考慮到有老人和小孩，預算在 X 元以內，並包含至少三個適合家庭的活動。」這可能需要一個「總策劃代理」來協調「航班搜尋代理」、「住宿預訂代理」、「活動推薦代理」以及「預算管理代理」。這些代理可以按順序或並行工作，共享資訊（例如透過共享的記憶或訊息傳遞），最終由主代理整合結果呈現給使用者。
    *   **專業任務分配：** 不同的代理可以擁有高度專業化的技能。例如，一個代理可能專精於分析財務報表，另一個擅長撰寫市場行銷文案，還有一個專門負責生成和測試程式碼。它們可以根據任務需求被動態地調用和組合。

2.  **管理多個並行對話 (Managing multiple concurrent conversations):**
    *   一個核心的「協調代理 (Orchestrator Agent)」或代理服務平台可能需要同時處理來自多個使用者的請求，或者同時管理多個正在進行的、由不同子代理執行的任務。代理框架需要提供有效的機制來管理這些並行的互動流程、狀態和資源。

3.  **代理間通訊 (Inter-agent communication):**
    *   代理之間需要一種標準化的方式來交換訊息、傳遞數據、請求服務和協調行動。框架通常會定義通訊協議和訊息格式，確保不同代理間可以順暢交流。

4.  **狀態管理與生命週期：**
    *   每個代理可能有其自身的內部狀態、目標和生命週期（創建、運行、暫停、終止）。框架需要提供工具來管理這些方面，確保系統的穩定性和可追蹤性。

**為何需要多代理系統？**

*   **模組化與可維護性：** 將龐大複雜的問題分解為更小、更易於管理和開發的獨立代理模組。
*   **專業化與效率：** 每個代理可以針對特定領域或任務進行深度優化，從而提高整體解決方案的效率和品質。
*   **可擴展性：** 當需求變化或增加時，可以更容易地添加新的專業代理或擴展現有代理的能力，而無需重構整個系統。
*   **強健性與容錯性：** 如果系統中的某個代理出現故障，其他代理可能仍能繼續運作，或者系統可以設計成能夠優雅地處理部分失敗，而不是完全崩潰。


**實作範例解析：**

```python
import semantic_kernel as sk
from semantic_kernel.functions import kernel_function, kernel_function_parameter
from typing import Annotated # 用於更清晰的類型提示 (Python 3.9+)

# --- 模擬 Agent 1: 旅行規劃插件 (TravelPlannerPlugin) ---
class TravelPlannerPlugin:
    """
    一個模擬的旅行規劃代理，可以為特定目的地規劃簡短行程。
    """
    @kernel_function(description="為給定的目的地城市規劃一個簡短的行程建議。")
    def plan_short_trip(self, destination: Annotated[str, "目的地城市"]) -> Annotated[str, "行程建議"]:
        if destination.lower() == "paris":
            return f"行程建議 ({destination}):\nDay 1: 參觀艾菲爾鐵塔與羅浮宮。\nDay 2: 漫步聖母院周邊與塞納河遊船。"
        elif destination.lower() == "tokyo":
            return f"行程建議 ({destination}):\nDay 1: 探索澀谷與新宿御苑。\nDay 2: 參觀淺草寺與晴空塔。"
        else:
            return f"抱歉，目前我只能為 Paris 和 Tokyo 提供簡短的行程規劃建議。"

# --- 模擬 Agent 2: 食譜搜尋插件 (RecipeFinderPlugin) ---
class RecipeFinderPlugin:
    """
    一個模擬的食譜搜尋代理，可以根據菜系尋找食譜。
    """
    @kernel_function(description="根據指定的菜系尋找一份推薦食譜。")
    def find_recipe(self, cuisine: Annotated[str, "菜系類型"]) -> Annotated[str, "食譜建議"]:
        if cuisine.lower() == "french":
            return f"法式食譜推薦：試試看製作一道簡易的『法式紅酒燉雞 (Coq au Vin)』。"
        elif cuisine.lower() == "italian":
            return f"義式食譜推薦：經典的『番茄羅勒義大利麵』是個好選擇。"
        else:
            return f"抱歉，目前我主要推薦 French 和 Italian 的食譜。"

# --- 主協調邏輯 (Orchestrator / Main Application Logic) ---
async def run_multi_specialist_scenario(kernel: sk.Kernel):
    """
    演示如何協調不同的專家插件 (模擬代理) 來處理一個多部分請求。
    """
    # 1. 將我們的專家插件 (模擬代理) 添加到 Kernel 中
    #    這樣 Kernel 就能夠找到並執行它們的功能。
    kernel.add_plugin(TravelPlannerPlugin(), plugin_name="TravelAgent")
    kernel.add_plugin(RecipeFinderPlugin(), plugin_name="ChefAgent")

    user_request = "你好，我想請你幫我規劃一個去巴黎的短程旅行，另外也想找一份法式晚餐的食譜。"
    print(f"使用者請求: {user_request}\n")

    # 2. 協調邏輯 (此處為簡化版：手動分解和路由任務)
    #    在一個更進階的多代理系統中，這一步可能由一個專門的 "Orchestrator Agent"
    #    或 Semantic Kernel 的 Planner 來自動決定調用哪些代理/技能。

    #    從使用者請求中提取關鍵資訊 (簡化)
    travel_destination = "Paris" # 假設我們已從請求中解析出目的地
    recipe_cuisine = "French"   # 假設我們已從請求中解析出菜系

    final_assistant_responses = []

    # 階段 1: 處理旅行規劃部分 (調用 TravelAgent)
    print(f"協調者：正在請求 TravelAgent 規劃 {travel_destination} 的行程...")
    try:
        travel_plan_result = await kernel.invoke_async(
            plugin_name="TravelAgent",
            function_name="plan_short_trip",
            arguments={"destination": travel_destination}
        )
        travel_response = str(travel_plan_result.value)
        print(f"TravelAgent 回應: {travel_response}\n")
        final_assistant_responses.append(f"關於您的 {travel_destination} 行程：\n{travel_response}")
    except Exception as e:
        print(f"TravelAgent 處理出錯: {e}\n")
        final_assistant_responses.append(f"抱歉，處理 {travel_destination} 行程規劃時發生問題。")


    # 階段 2: 處理食譜搜尋部分 (調用 ChefAgent)
    print(f"協調者：正在請求 ChefAgent 尋找 {recipe_cuisine} 食譜...")
    try:
        recipe_result = await kernel.invoke_async(
            plugin_name="ChefAgent",
            function_name="find_recipe",
            arguments={"cuisine": recipe_cuisine}
        )
        recipe_response = str(recipe_result.value)
        print(f"ChefAgent 回應: {recipe_response}\n")
        final_assistant_responses.append(f"關於您的 {recipe_cuisine} 食譜：\n{recipe_response}")
    except Exception as e:
        print(f"ChefAgent 處理出錯: {e}\n")
        final_assistant_responses.append(f"抱歉，處理 {recipe_cuisine} 食譜搜尋時發生問題。")

    # 3. 組合最終回應給使用者
    #    這個組合後的回應可以被添加到 ChatHistory 中，如第一個範例所示。
    print("--- 整合後的 AI 助手回應 ---")
    full_response = "\n\n".join(final_assistant_responses)
    print(full_response)

```


Semantic Kernel 的核心設計理念，如插件化架構（允許每個代理擁有不同的工具集）、規劃器 (Planner)（可以幫助代理決定行動步驟，甚至選擇其他代理協作）、以及記憶 (Memory)（可以作為代理間共享知識的基礎），本身就為構建更複雜的代理系統奠定了堅實的基礎。

其專用的代理框架會在此之上提供更高級別的抽象和工具，例如：
*   標準化的代理接口 (Agent Interface) 定義。
*   群聊管理 (Group Chat Management) 功能，允許多個代理（甚至可以包含人類使用者）在同一個對話上下文中進行互動和協作。
*   代理的創建、註冊、發現和生命週期管理機制。