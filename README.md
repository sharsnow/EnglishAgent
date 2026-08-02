# EnglishAgent 專案報告

> EnglishAgent 是一個整合 AWS、AI 與 RAG 的英文學習平台。
> 本專案結合雲端部署、知識庫檢索與多代理流程，提供可互動的英文教學與問答體驗。

## 1. 專案簡介

EnglishAgent 的核心目標，是把英文學習、教材查詢、對話式回覆與雲端服務整合到同一個平台中。  
使用者可以透過網頁登入後進入聊天介面，向系統提出英文學習問題，系統會依照問題內容啟動對應的 AI / RAG 流程，從 PDF 教材與知識庫中檢索資訊，再產生有教學脈絡的回覆。

這個專案同時展示了：

- AWS 雲端基礎架構的設計
- AI 模型與 RAG 的整合方式
- Flask 後端與網頁前端的串接
- 使用 RDS、S3、ChromaDB、Bedrock 建立完整的應用流程

## 2. 專案目標

| 項目 | 說明 |
|---|---|
| 雲端化部署 | 以 AWS 建立穩定、可擴充、可維運的服務環境。 |
| AI 教學 | 提供可互動的英文學習與問答體驗。 |
| 知識整合 | 將 PDF 教材與專案知識轉為可檢索資料。 |
| 高可用架構 | 使用 ALB、ASG、RDS 與 VPC 提升服務穩定性。 |
| 模組化設計 | 讓前端、後端、知識庫與模型推論可以獨立演進。 |

## 3. 雲端架構

### 3.1 架構概觀

EnglishAgent 的雲端架構可以分成五個主要層次：

1. 使用者存取層
2. 前端與應用層
3. AI / RAG 推論層
4. 資料與儲存層
5. 安全與維運層

整體資料流為：

```text
User -> ALB -> EC2 / Flask Backend -> Master Agent -> RAG / Bedrock -> Response -> RDS / Logs
```

### 3.2 使用者存取層

使用者會先透過 Application Load Balancer 的 DNS URL 進入系統。  
從畫面上可以看到，登入頁直接對外提供存取入口，代表實際服務已經透過 ALB 做公開暴露，而不是直接將 EC2 對外開放。

這樣做的好處是：

- 將外部流量集中到單一入口
- 方便未來切換後端實例
- 提升安全性與維護彈性
- 可以搭配憑證與健康檢查機制

### 3.3 前端與應用層

目前專案前端以 HTML + JavaScript 為主，後端則以 Flask RESTful API 提供服務。  
使用者登入後會進入聊天頁面，左側是功能選單，中央是對話內容區，底部是輸入框。

應用層的職責包括：

- 驗證使用者身份
- 接收使用者輸入
- 轉送請求給 AI / RAG 模組
- 呈現回覆與學習內容
- 顯示歷史訊息與教學分類

### 3.4 AI / RAG 推論層

AI 層是 EnglishAgent 的核心。  
專案中使用 Amazon Bedrock 作為模型存取入口，搭配不同模型處理不同任務：

- Claude 3.5 Sonnet：負責較深度的推理與教學回覆
- Claude 3 Haiku：負責較快速的反應與輕量任務
- Titan Embeddings V2：負責將教材轉成向量表示

推論層採用 Master-Sub Agent 或任務分工概念，把使用者問題拆成：

- 意圖判斷
- 內容檢索
- 教學整理
- 最終回覆生成

### 3.5 資料與儲存層

專案在資料面主要使用以下元件：

| 元件 | 用途 |
|---|---|
| Amazon S3 | 儲存 PDF 教材、靜態資源與原始知識來源。 |
| Amazon RDS (MySQL) | 儲存使用者、聊天歷史與系統狀態資料。 |
| ChromaDB | 儲存向量索引，供 RAG 檢索使用。 |

這個設計的好處是把「原始文件」、「結構化資料」與「向量知識」分開管理，避免所有資料都塞在同一層，方便日後擴充與維護。

### 3.6 安全與維運層

雲端架構還包含多個基礎安全元件：

- VPC：建立網路隔離邊界
- Security Group：限制服務間的連線
- IAM Role：讓 EC2 以角色方式存取 Bedrock、S3 等 AWS 服務
- Secrets Manager：可用於保管密碼或 API 金鑰
- Auto Scaling Group：依流量調整執行個體數量
- GitHub：作為版本控管與部署流程的一部分

這些元件讓系統不只是能跑，也能在實際環境中持續運作。

## 4. 架構元件說明

| 元件 | 說明 |
|---|---|
| Amazon EC2 | 執行 Flask 後端、AI 邏輯與 API 服務。 |
| Application Load Balancer | 接收外部流量並分派到健康的 EC2 實例。 |
| Auto Scaling Group | 根據負載擴縮 EC2，提升可用性。 |
| Amazon VPC | 隔離公私網路並控制子網與路由。 |
| Amazon S3 | 存放 PDF 教材與專案資源。 |
| Amazon RDS (MySQL) | 保存使用者資料、聊天紀錄與系統資訊。 |
| ChromaDB | 儲存教材向量，支援語意檢索。 |
| Amazon Bedrock | 提供 Claude 與 Titan 等模型能力。 |
| IAM / Security Group | 控制 AWS 資源權限與網路存取。 |
| Secrets Manager | 儲存敏感設定，避免硬編碼。 |

## 5. 資料流程

### 5.1 使用者進入系統

1. 使用者打開 ALB 的 DNS 網址。
2. 進入登入頁面，輸入帳號密碼。
3. 驗證完成後，導向聊天主頁。

### 5.2 互動與推論流程

1. 使用者輸入英文學習需求或問題。
2. Flask 後端接收請求並解析意圖。
3. Master Agent 決定是否需要查詢知識庫。
4. 若需要檢索，系統會將查詢向量化。
5. ChromaDB 找出與問題最相關的教材片段。
6. Bedrock 上的 LLM 根據檢索結果產生回答。
7. 回答與學習歷程寫回 RDS。

### 5.3 簡化流程圖

```text
User
  -> ALB
  -> EC2 / Flask
  -> Master Agent
  -> RAG / Bedrock
  -> Response
  -> RDS / Logs
```

## 6. Agent 與 AI 設計

### 6.1 Agent 設計理念

EnglishAgent 採用 Master-Sub Agent 的概念，讓不同任務由不同角色處理：

- Master Agent：理解使用者目標、決定流程
- Question Agent：整理問題與意圖
- Teaching Agent：產生教學內容
- Answer Agent：生成最終回答

這種方式的優點是：

- 邏輯清楚
- 每個 agent 負責單一職責
- 容易擴充更多教學功能
- 可以降低單一 prompt 過長造成的不穩定

### 6.2 模型使用方式

| 模型 / 機制 | 用途 |
|---|---|
| Claude 3.5 Sonnet | 複雜推理、教學整理、較高品質的回覆。 |
| Claude 3 Haiku | 快速反應、輕量分類與簡單回答。 |
| Titan Embeddings V2 | 產生向量，用於語意檢索。 |
| RAG | 從教材中找出相關段落作為回答依據。 |
| Chain-of-Thought | 協助模型在內部整理推理路徑。 |

## 7. RAG 與知識庫

### 7.1 知識來源

EnglishAgent 的知識來源主要包含：

- 英文教材 PDF
- 題庫或學習筆記
- 系統內部的課程內容
- 對話歷程與使用者行為資料

### 7.2 RAG 流程

1. 讀取 PDF 或教材內容。
2. 將內容切塊，避免單段文字過長。
3. 透過 Titan Embeddings V2 轉成向量。
4. 存入 ChromaDB 建立索引。
5. 使用者提問時，先做語意檢索。
6. 將檢索到的內容交給 LLM 生成最終答案。

### 7.3 RAG 的價值

RAG 的目的不是單純讓 AI「自己回答」，而是讓 AI 先參考專案知識再回答。  
這能提升：

- 正確性
- 可追溯性
- 教材一致性
- 對 PDF 文件內容的對應能力

## 8. 7 張圖說明

以下 7 張圖對應的是專案報告與雲端架構的核心內容。

### 8.1 系統架構圖

![系統架構](assets/figure_01_architecture.png)

這張圖是整體架構總覽。可以看到使用者先經過 ALB 進入系統，流量再進到 VPC 內的 EC2 與 Flask 後端。  
圖中也標示了 Bedrock、S3、RDS 與 ChromaDB 的關係，說明 EnglishAgent 不是單一聊天頁面，而是一套完整的雲端應用：

- 前端負責接收操作
- 後端負責流程控制
- Bedrock 負責 AI 推理
- S3 提供教材來源
- RDS 儲存狀態與歷史
- ChromaDB 支援 RAG 檢索

### 8.2 雲端堆疊圖

![雲端堆疊](assets/figure_02_cloud_stack.png)

這張圖把系統拆成幾個層級來看：

- User Access
- Frontend
- Application Tier
- Data Tier
- Security & Integration
- DevOps

它特別說明了：

- 前端目前是 HTML + JavaScript
- 後端是 Flask RESTful API
- S3 可作為靜態資源與教材儲存位置
- ALB 是 API 統一入口
- ASG 會依 CPU 或負載進行擴縮
- RDS 作為結構化資料儲存
- IAM、Security Group、VPC、Secrets Manager 則負責安全與整合

### 8.3 登入畫面

![登入畫面](assets/figure_03_login.png)

這張圖是系統的入口頁。使用者會透過 ALB 的 DNS URL 進入登入介面，輸入 StudentID 與 Password 後再進入主系統。  
它代表：

- 服務是透過雲端公開入口提供
- 前端有基本的身份驗證流程
- 系統不是直接開放後台，而是先經過登入入口

### 8.4 聊天主畫面

![聊天主畫面](assets/figure_04_chat.png)

這張圖展示 EnglishAgent 的主要互動介面：

- 左側是功能選單
- 中間是對話內容
- 底部是使用者輸入框
- 左下角顯示使用者資訊

畫面上也可看到歷史訊息是從 Amazon RDS 即時讀取，這表示聊天內容與使用者狀態並非只存在前端，而是有後端資料庫支援。

### 8.5 RAG 對話畫面

![RAG 對話](assets/figure_05_rag_chat.png)

這張圖是 RAG 的實際應用示意。當使用者提出英文學習問題時，系統會先從 ChromaDB 取回與教材相關的向量片段，再把這些內容交給 Claude 類模型生成回覆。

圖中也強調了這個流程：

- Titan Embeddings 用於建立向量
- ChromaDB 用於檢索
- Claude 3.5 Sonnet / Claude 3 Haiku 用於產生回答

這是 EnglishAgent 最重要的教學能力之一，因為它讓回答更貼近教材內容，而不是純粹的自由生成。

### 8.6 Multi-AZ 與目標群組畫面

![Multi-AZ 架構](assets/figure_06_multi_az.png)

這張圖是 AWS 負載平衡與健康檢查的管理畫面，重點在於：

- Target Group 目前註冊了可運作的後端實例
- 健康狀態為 Healthy
- 目標埠號是 5000
- 區域顯示在 ap-southeast-1 類型的環境中

這代表 EnglishAgent 不只是單機部署，而是採用可擴充、可監控的方式運作。  
搭配 Multi-AZ 或 ASG 設計後，當某一台 EC2 故障時，流量可以自動導向其他健康節點。

### 8.7 心智圖

![心智圖](assets/figure_07_mind_map.png)

這張圖整理了 EnglishAgent 的整體概念與功能區塊，從中心的 EnglishAgent Cloud 出發，延伸到：

- AWS Cloud Infrastructure
- Web Architecture
- Database & Storage
- Core Agent Roles
- AI / LLM
- RAG System

它也把各層的細節拆出來，例如：

- AWS 端包含 VPC、ALB、EC2、ASG、S3、RDS
- AI 端包含 Claude、Titan Embeddings、Amazon Bedrock
- 資料端包含 RDS、ChromaDB、S3
- Web 端包含前端與 Flask API

這張圖很適合作為專案總結頁，因為它把整個專案的技術範圍一次收斂到一張圖裡。

## 9. 系統特色與限制

| 項目 | 說明 |
|---|---|
| 優點 | 整合雲端、AI、RAG 與教材管理，結構完整。 |
| 優點 | 可用模組化方式擴充更多任務與教學功能。 |
| 優點 | 使用 AWS 元件，具備較好的可用性與維運彈性。 |
| 限制 | PDF 與知識庫需要持續更新，否則答案可能過時。 |
| 限制 | 模型回應速度會受推論與檢索流程影響。 |
| 限制 | prompt 與 agent 路由需要持續調整才能穩定。 |

## 10. 專案目錄與執行

### 10.1 常用目錄

| 目錄 | 說明 |
|---|---|
| `pages/` | 靜態網頁入口，包含 `index.html`。 |
| `assets/` | 圖片與簡報素材。 |
| `README.md` | 專案說明文件。 |

### 10.2 啟動方式

```bash
pip install -r requirements.txt
python run.py
```

### 10.3 AWS 部署提醒

- PDF 教材建議放在 Amazon S3 管理。
- ALB 是對外入口，適合掛在前面做流量分配。
- EC2 建議使用 IAM Role 來存取 AWS 服務，不要硬編碼金鑰。
- RDS 建議搭配安全群組與私有網路。
- ChromaDB 可作為 RAG 的向量資料庫。

## 11. 後續維護建議

- 定期更新教材 PDF 與向量索引。
- 監控 RDS、EC2、ALB 與 ASG 健康狀態。
- 優化 prompt 與 agent 分工，讓回答更穩定。
- 若未來流量增加，可將前端改為 S3 靜態網站或 CDN 架構。
- 持續補充更多題型、更多學習情境與更完整的回饋機制。

