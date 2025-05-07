# 記憶與向量存儲 (Memory and Vector Storage)

想像一下，如果與您對話的 AI 每次都像初次見面一樣，完全不記得之前的交流內容或您的偏好，那樣的體驗想必不會太好。為了解決這個問題，Semantic Kernel 引入了強大的**記憶 (Memory)** 功能，讓 AI 能夠「記住」資訊，從而提供更連貫、更個性化、更智能的回應。而這背後的關鍵技術之一，便是**向量儲存 (Vector Storage)**。

## 什麼是 AI 的「記憶」？

在 Semantic Kernel 的語境下，「記憶」並非指 AI 模型本身的權重更新或重新訓練。相反地，它指的是一種**外部化的、可供 AI 隨時存取和檢索的資訊儲存庫**。

這個記憶庫可以儲存各種資訊，例如：

*   **過去的對話片段：** 讓 AI 記得與特定使用者的互動歷史。
*   **使用者偏好：** 例如，使用者喜歡簡潔的回答，或者偏愛某種風格的內容。
*   **特定領域知識：** 您可以將產品文件、常見問題解答 (FAQ) 或任何專業知識「餵」給記憶體，讓 AI 能夠基於這些資訊回答問題。
*   **臨時上下文：** 在一個多步驟的任務中，AI 可以將中間結果存入記憶，供後續步驟使用。

**核心目標：** 讓 AI 能夠在需要時，快速找到並利用相關的歷史資訊或知識，以生成更貼切、更有深度的回應。

## 記憶的運作核心：向量儲存 (Vector Storage)

傳統的資料庫通常依賴關鍵字匹配來搜尋資訊。但 AI 的「記憶」需要更進階的方式——它需要理解語義上的相似性。這就是**向量儲存**大顯身手的地方。

1.  **文字轉換為向量 (Embeddings)：**
    當您想要將一段文字 (例如一句話、一個段落或一份文件) 存入記憶時，Semantic Kernel (或其底層的 AI 模型) 會先將這段文字轉換成一串數字，稱為「向量嵌入 (Vector Embedding)」。這個向量可以被認為是該文字在多維空間中的一個「語義座標」，能夠捕捉文字的意義和上下文。意義相近的文字，其向量在空間中的距離也會比較接近。

2.  **儲存於向量資料庫：**
    這些生成的向量會被儲存在專門的**向量資料庫 (Vector Database)** 或支援向量搜尋的資料庫中。這些資料庫經過優化，能夠高效地儲存大量向量，並快速執行相似性搜尋。

3.  **語義檢索 (Semantic Search)：**
    當 AI 需要從記憶中檢索資訊時（例如，為了回答一個使用者的問題），它會：
    *   將使用者的問題（或當前對話的上下文）也轉換成一個向量。
    *   然後在向量資料庫中搜尋與這個「查詢向量」最相似（即距離最近）的已儲存向量。
    *   找到最相似的向量後，系統會取回它們對應的原始文字內容。

這整個過程稱為**語義檢索 (Semantic Retrieval)** 或**上下文檢索 (Context Retrieval)**，因為它不是基於簡單的字詞匹配，而是基於意義的理解。

## Semantic Kernel 如何整合向量資料庫？

Semantic Kernel 的設計極具擴展性，它提供了**向量儲存連接器 (Vector Storage Connectors)**，使開發者能夠輕鬆地將各種主流的向量資料庫或服務整合到他們的 AI 應用中。

*   **Azure AI Search (formerly Azure Cognitive Search):** 微軟提供的強大雲端搜尋服務，支援向量搜尋。
*   **Elasticsearch:** 一個廣泛使用的開源搜尋和分析引擎，也具備向量搜尋能力。
*   **Chroma:** 一個專為 AI 應用設計的開源嵌入式資料庫。
*   **Qdrant, Pinecone, Weaviate 等：** 還有許多其他流行的向量資料庫也都有相應的連接器或可以被整合。

此外，Semantic Kernel 也提供了一個**內存中的記憶體儲存 (VolatileMemoryStore)**，它不需要連接外部資料庫，直接在程式的記憶體中運作。這對於快速原型開發、測試或小型應用非常方便，但程式結束後資料就會遺失。

## 範例: 在 Semantic Kernel 中實際操作記憶

```python
# 假設 'kernel' 是一個已經初始化並配置好的 Semantic Kernel 實例。
# import semantic_kernel as sk
# from semantic_kernel.memory import VolatileMemoryStore # 引入內存記憶體儲存

# kernel = sk.Kernel()
# ... (此處省略 kernel 初始化及 AI 服務配置的相關程式碼) ...

# 1. 初始化一個記憶體儲存實例
#    這裡使用的是 VolatileMemoryStore，它將數據儲存在程式的RAM中。
#    對於生產環境，您可能會替換成 AzureAISearchMemoryStore 或 ChromaMemoryStore 等。
memory_store = VolatileMemoryStore()

# 2. 將記憶體儲存實例作為一個服務添加到 Kernel 中
#    這樣 Kernel 就能夠使用這個記憶體儲存來執行保存和檢索操作。
kernel.add_service(memory_store)

# 3. 儲存資訊到記憶體中
#    使用 kernel.memory 來訪問記憶體功能。
#    save_information_async 方法用於異步保存一條資訊。
await kernel.memory.save_information_async(
    collection="user_data",
    id="user1", # 根據您提供的範例，ID 為 "user1"
    text="User prefers dark mode" # 根據您提供的範例
)
```

### 記憶帶來的好處

*   **更相關的回應：** AI 可以參考過去的互動和已儲存的知識，避免重複詢問或提供與當前對話無關的資訊。
*   **個性化體驗：** 透過記住使用者的偏好、歷史行為或特定需求，AI 可以提供更加量身定制的服務和互動。
*   **長程對話能力：** 在持續多輪的對話中，記憶幫助 AI 保持對話的上下文連貫性，理解指代關係，並追蹤對話的進程。
*   **外部知識整合：** 讓 AI 能夠「學習」並利用您的私有數據、公司文件、專業知識庫等，極大地擴展其知識邊界。
*   **減少對 LLM 的重複請求與成本：** 對於相對靜態或常用的資訊，可以先從本地或快速的向量記憶中檢索，如果找到合適的內容，可能就不需要每次都請求昂貴的 LLM 進行生成或回答。
