# Multi-Agent 永續議題辯論系統

## 一、 專案簡介
這是一個基於 LangChain 與 LangGraph 建構的多重代理人（Multi-Agent）辯論系統。本專案以「永續發展（ESG）」為核心議題，模擬不同政治與經濟立場的代理人進行交互辯論。系統結合了 RAG（檢索增強生成）技術與 Tavily 網路搜尋，使代理人能根據內部專屬知識庫與最新網路資訊，提出具體數據佐證其論點，並在特定的辯論輪數後，引導人類使用者參與討論與決策。

## 二、 系統架構與核心技術
1. **流程框架與模型**：使用 LangChain 與 LangGraph 管理對話狀態（State）與節點路由（Routing），底層使用 OpenAI 的語言模型與嵌入模型。
2. **檢索增強生成（RAG）**：使用 `langchain_community.vectorstores.Chroma` 建立本地向量資料庫，儲存專案相關的政策與行動方案 PDF 文件。
3. **即時網路搜尋**：整合 Tavily API 進行即時網路資料檢索，確保代理人在辯論時能引用最新的國際趨勢與事實佐證。
4. **狀態與記憶管理**：使用 `langgraph.checkpoint.memory.InMemorySaver` 與 `StateGraph` 記錄對話狀態、訊息歷史以及當前發言的代理人標籤。

## 三、 核心代理人角色設定
系統中定義了三個主要節點，彼此透過條件路由進行切換：

1. **主持人（Operator Agent）**：
   * **主要職責**：引導辯論主題，並透過 Tavily 搜尋建立具爭議性的永續話題。負責控制流程，決定將發言權轉交給 Agent A、Agent B 或是人類使用者。
   * **互動規則**：設定在辯論 2 到 3 輪後，主動整理前面討論重點並引導使用者發表看法。必須等使用者多次表達明確立場後，才能總結並結束對話。
2. **Agent A（賴清德立場）**：
   * **核心立場**：積極推動淨零轉型、發展綠能與數位雙軸轉型、強調供應鏈接軌國際 ESG 標準。
   * **資料來源**：優先檢索 `taiwan_climate_docs` 向量資料庫（包含台灣官方的環保與減碳政策文件），並輔以網路搜尋。
3. **Agent B（川普立場）**：
   * **核心立場**：反對過度碳稅與激進綠能政策、保護本土產業與勞工、強調經濟發展優先。
   * **資料來源**：優先檢索 `trump_need` 向量資料庫（包含 `trump_file.pdf` 等相關文件），並輔以網路搜尋。

## 四、 檔案與資料夾結構
* `main.ipynb`：系統主程式。定義各代理人的系統提示詞（System Prompt）、自訂工具（本地檢索與網路搜尋）以及 LangGraph 的狀態機器路由邏輯。
* `load.ipynb`：資料前處理與知識庫建置程式。負責讀取台灣氣候與減碳相關的 PDF 文件，使用 `RecursiveCharacterTextSplitter` 進行文本分割，並存入 Chroma 向量資料庫。
* `MultiAgentTest.ipynb`：多重代理人測試腳本。這是一個簡易的銷售與客服代理人切換範例，用於初步驗證 LangGraph 的路由與交接（Handoff）機制。
* `taiwan_climate_docs/`：存放 Agent A 專用的 Chroma 本地向量資料庫。
* `trump_need/`：存放 Agent B 專用的 Chroma 本地向量資料庫。
* `*.pdf`：各類參考與知識庫來源文件（例如：《台灣總體減碳行動計畫.pdf》、《深度節能行動方案.pdf》、《空品政策白皮書.pdf》等）。
* `Debate Demo.mov` / `Debate_Demo1.mp4`：系統執行結果的展示影片檔。
* `langgraph.png`：LangGraph 狀態機器的圖形化架構圖。
* `.env`：環境變數設定檔（存放 OpenAI 與 Tavily 的 API Keys）。

## 五、 建置與執行說明
1. **環境變數設定**：在專案根目錄建立（或修改） `.env` 檔案，確保已填寫必要的 API 金鑰：
   ```env
   OPENAI_API_KEY=your_openai_api_key
   TAVILY_API_KEY=your_tavily_api_key
   ```
2. **建立專屬知識庫**：
   * 執行 `load.ipynb` 中的儲存格，系統會自動讀取指定的 PDF 文件並建立 `taiwan_climate_docs` 向量資料庫。
   * （視專案需求確認 `trump_need` 資料庫已建構完成）。
3. **執行主程式進行辯論**：
   * 執行 `main.ipynb`。
   * 系統啟動後，將由主持人起頭建立辯論題目，並交由兩位代理人交替發言。代理人會在發言前自動調用工具搜尋資料。
   * 根據對話進度，系統會於終端機提示使用者參與互動（顯示 `請輸入你的觀點或立場:`），使用者可藉此表達對該永續議題的看法，影響後續對話走向。
# sustainability_debate
