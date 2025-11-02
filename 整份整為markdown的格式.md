# **社區關懷送餐系統 \- 產品需求文件 (PRD)**

## **1\. 系統概述 (System Overview)**

**專案目標：** 解決社區送餐的「最後一哩路」問題。本系統旨在透過技術輔助，確保餐點準時、準確地送達個案手中，並提供即時的服務回報與追蹤機制。

**核心價值：**

* **提升效率：** 透過路線規劃與導航，縮短送餐時間，簡化志工交接流程。  
* **確保品質：** 透過 QR Code 驗證與強制拍照回報，確保服務確實執行。  
* **即時監控：** 讓管理者能即時掌握送餐進度，主動處理異常狀況。  
* **強化關懷：** 透過特殊備註與回報機制，傳達個案的即時需求，實現真正的社區關懷。

## **2\. 使用者角色與情境 (User Personas)**

| 角色 | 平台 | 核心情境 |
| :---- | :---- | :---- |
| **管理者 (Admin)** | Web 後台 | 負責建立個案資料、規劃每日送餐路線、指派送餐員、即時監控所有路線進度，並查閱歷史回報紀錄。 |
| **送餐員 (Driver)** | Mobile App | 負責接收當日任務、依序前往個案地點、透過 App 導航、掃描 QR Code 並拍照回報送餐狀態。 |

## **3\. 技術架構 (Technical Architecture)**

| 層級 (Layer) | 選用技術 | 目的 |
| :---- | :---- | :---- |
| **管理員後台 (Admin)** | **React** (搭配 Vite, TailwindCSS) | 提供高效能、高互動性的單頁應用程式 (SPA) 介面。 |
| **送餐員 App (Mobile)** | **React Native** (或 Flutter) | 一次開發，雙平台 (iOS/Android) 部署，並能存取原生相機、GPS 與離線儲存。 |
| **後端 (Backend)** | **Node.js** (搭配 Express/NestJS) | 處理高併發 I/O，適合 RESTful API 與 WebSocket 即時通訊。 |
| **資料庫 (Database)** | **PostgreSQL** (GCP Cloud SQL) | 強大的關聯式資料庫，並可搭配 PostGIS 進行地理位置查詢。 |
| **圖片儲存 (Storage)** | **GCP Cloud Storage** | 儲存送餐員回報的照片，高可用且成本低廉。 |
| **地理位置 (Geolocation)** | **WebSockets** \+ **Google Maps Platform** | WebSockets 用於即時位置回報；Google Maps API 用於路線優化與導航。 |
| **認證 (Authentication)** | **JWT** (JSON Web Tokens) | 無狀態 (Stateless) 的認證機制，適用於 Web 和 Mobile。 |

## **4\. 功能需求 (Functional Requirements)**

### **4.1. 管理員後台 (Admin Web)**

* **FR-A1: 登入 (Login)**  
  * 管理者必須使用帳號密碼登入系統。  
* **FR-A2: 儀表板 (Dashboard)**  
  * **FR-A2.1 (即時地圖):** 提供一個全螢幕地圖，即時顯示所有「送餐中」的送餐員 GPS 位置。  
  * **FR-A2.2 (進度總覽):** 以列表或卡片形式，顯示當日所有路線的總覽，包含：路線名稱、負責送餐員、進度（例如：已完成 5/12 站）。  
* **FR-A3: 個案管理 (Recipient Management \- CRUD)**  
  * **FR-A3.1 (資料管理):** 建立、讀取、更新、刪除個案資料，欄位包含：姓名、地址、電話、經緯度（地址自動轉換）、**特殊備註**（例如：素食、糖尿病、行動不便等）。  
  * **FR-A3.2 (QR Code):** 系統必須為每一個案**生成一個唯一的 QR Code**，並提供「列印」功能，以便管理者將其貼於個案家門口。  
* **FR-A4: 路線規劃 (Route Planning)**  
  * **FR-A4.1 (每日任務):** 管理者能建立「每日送餐任務」（例如：2025-11-02 路線 A）。  
  * **FR-A4.2 (指派任務):** 能將建立的路線任務，指派給特定的送餐員。  
  * **FR-A4.3 (站點排序):** 能從「個案列表」將多個個案加入一條路線，並支援**拖曳排序**。  
  * **FR-A4.4 (路徑優化):** (可選) 提供「最佳路徑」按鈕，呼叫 Google Maps Directions API 自動規劃最短送餐順序。  
* **FR-A5: 紀錄查詢 (Monitoring & Reporting)**  
  * 管理者能依日期、路線、送餐員查詢歷史送餐紀錄。  
  * 能查看每一站的詳細資料，包含：個案姓名、送達時間、**回報照片**、送餐員備註（例如：本人領取、無人應門）。

### **4.2. 送餐員 App (Driver Mobile)**

* **FR-D1: 登入 (Login)**  
  * 送餐員必須使用帳號密碼登入。App 應記住登入狀態。  
* **FR-D2: 我的任務 (My Route)**  
  * **FR-D2.1 (任務列表):** 登入後，首頁即顯示當天被指派的路線列表（需依照管理者排定的順序）。  
  * **FR-D2.2 (個案資訊):** 列表中清楚顯示個案姓名、地址、電話，並**醒目提示特殊備註**（如圖片範例）。  
  * **FR-D2.3 (一鍵導航):** 每個站點提供「導航」按鈕，點擊後呼叫手機內建地圖 (Google Maps / Apple Maps) 開始導航。  
* **FR-D3: 送達回報 (Confirmation Workflow)**  
  * **FR-D3.1 (掃碼驗證):** 抵達後，點擊「掃描 QR Code」按鈕。App 掃描個案家門口的 QR Code，並傳送至後端 API 進行驗證。  
  * **FR-D3.2 (強制拍照):** QR Code 驗證成功後，App **必須自動開啟相機**，送餐員需拍照（例如：餐點與門牌）以茲證明。**禁止**從相簿上傳照片。  
  * **FR-D3.3 (簡易備註):** (可選) 拍照後，提供簡易選項（例如：本人領取、家人代領、無人應門）及文字備註欄。  
  * **FR-D3.4 (提交完成):** 點擊「提交」後，該站點在任務列表上標示為「已完成」（如圖片範例）。  
* **FR-D4: 離線支援 (Offline Mode)**  
  * 在網路訊號不佳或離線時：  
  * App 應能正常執行 FR-D3 (掃碼、拍照、備註)。  
  * 所有回報資料（包含照片檔案）應**暫存於手機本地儲存**。  
  * 一旦偵測到網路連線恢復，App 必須在背景**自動同步**所有暫存資料至後端伺服器。  
* **FR-D5: 即時定位 (Real-time Geolocation)**  
  * App 在「送餐中」狀態下，應定時（例如：每 30 秒）將當前 GPS 經緯度透過 WebSocket 或 API 回報至後端，供儀表板 (FR-A2.1) 使用。

## **5\. 非功能需求 (Non-Functional Requirements)**

| 類別 | 需求 |
| :---- | :---- |
| **效能 (Performance)** | App 啟動時間應 \< 3 秒。QR Code 掃描辨識應 \< 1 秒。 |
| **安全性 (Security)** | 個案資料（姓名、地址、電話、病史備註）皆屬 PII，傳輸過程 (HTTPS) 與儲存（資料庫加密）皆須加密。 |
| **易用性 (Usability)** | **(高優先級)** 送餐員 App 介面必須極簡、直觀。字體需加大、按鈕需明確，確保長輩志工也能輕鬆使用。 |
| **可靠性 (Reliability)** | 離線支援 (FR-D4) 必須可靠，確保資料不遺失。 |
| **合規性 (Compliance)** | API 需處理圖片上傳，儲存服務 (GCS) 需設定正確的存取權限 (CORS)。 |

## **6\. 資料模型 (Data Models \- PostgreSQL)**

```python
\-- 使用者 (管理員 & 送餐員)  
CREATE TABLE "Users" (  
  "id" SERIAL PRIMARY KEY,  
  "username" VARCHAR(50) UNIQUE NOT NULL,  
  "password\_hash" VARCHAR(255) NOT NULL,  
  "full\_name" VARCHAR(100),  
  "role" VARCHAR(10) NOT NULL CHECK ("role" IN ('ADMIN', 'DRIVER')), \-- 角色  
  "created\_at" TIMESTAMPTZ DEFAULT now()  
);

\-- 個案 (送餐對象)  
CREATE TABLE "Recipients" (  
  "id" SERIAL PRIMARY KEY,  
  "name" VARCHAR(100) NOT NULL,  
  "address" VARCHAR(255) NOT NULL,  
  "latitude" DECIMAL(10, 8),  
  "longitude" DECIMAL(11, 8),  
  "phone" VARCHAR(20),  
  "special\_notes" TEXT, \-- 特殊備註 (素食、糖尿病等)  
  "qr\_code\_id" VARCHAR(255) UNIQUE NOT NULL, \-- 用於驗證的唯一 ID  
  "is\_active" BOOLEAN DEFAULT true,  
  "created\_at" TIMESTAMPTZ DEFAULT now()  
);

\-- 路線任務 (每日任務)  
CREATE TABLE "Routes" (  
  "id" SERIAL PRIMARY KEY,  
  "name" VARCHAR(100) NOT NULL,  
  "assigned\_driver\_id" INT REFERENCES "Users"("id"), \-- 指派的送餐員  
  "route\_date" DATE NOT NULL,  
  "status" VARCHAR(20) DEFAULT 'PENDING' CHECK ("status" IN ('PENDING', 'IN\_PROGRESS', 'COMPLETED')),  
  "created\_at" TIMESTAMPTZ DEFAULT now()  
);

\-- 路線站點 (路線中的每一個送餐點)  
CREATE TABLE "RouteStops" (  
  "id" SERIAL PRIMARY KEY,  
  "route\_id" INT NOT NULL REFERENCES "Routes"("id") ON DELETE CASCADE,  
  "recipient\_id" INT NOT NULL REFERENCES "Recipients"("id"),  
  "sequence\_order" INT NOT NULL, \-- 送餐順序  
  "status" VARCHAR(20) DEFAULT 'PENDING' CHECK ("status" IN ('PENDING', 'COMPLETED', 'FAILED')),  
  "estimated\_delivery\_time" TIMESTAMPTZ, \-- (可選)  
  "created\_at" TIMESTAMPTZ DEFAULT now()  
);

\-- 送餐紀錄 (App 回報的最終紀錄)  
CREATE TABLE "DeliveryRecords" (  
  "id" SERIAL PRIMARY KEY,  
  "route\_stop\_id" INT UNIQUE NOT NULL REFERENCES "RouteStops"("id"), \-- 確保一站只有一筆紀錄  
  "delivery\_time" TIMESTAMPTZ NOT NULL,  
  "photo\_url" VARCHAR(512) NOT NULL, \-- GCS 上的照片 URL  
  "notes" TEXT, \-- 備註 (例如：家人代領)  
  "status" VARCHAR(50) NOT NULL, \-- (例如：DELIVERED\_SELF, DELIVERED\_FAMILY, ANOMALY\_NO\_ANSWER)  
  "created\_at" TIMESTAMPTZ DEFAULT now()  
);
```

## **7\. 系統架構圖 (GCP Deployment Architecture)**

```mermaid
graph TB  
    subgraph "使用者 (Clients)"  
        Admin\[管理員 (Web Browser)\]  
        Driver\[送餐員 (Mobile App)\]  
    end

    subgraph "Google Cloud Platform (GCP)"  
        LB\[Cloud Load Balancer\]

        subgraph "前端服務 (Frontend Services)"  
            CDN\[Cloud CDN\]  
            WebStorage\[Cloud Storage (Web 靜態檔案)\]  
        end

        subgraph "後端服務 (Backend Services)"  
            ApiServer\[Cloud Run (Node.js API 伺服器)\]  
            WebSocketServer\[Cloud Run (Node.js WebSocket 伺服器)\]  
        end

        subgraph "資料儲存 (Data Storage)"  
            DB\[Cloud SQL (PostgreSQL)\]  
            ImageStorage\[Cloud Storage (照片儲存)\]  
        end

        subgraph "GCP APIs"  
            MapsAPI\[Google Maps Platform\<br/\>(Geocoding, Directions)\]  
        end  
    end

    %% 流程  
    Admin \-- HTTPS \--\> LB  
    Driver \-- HTTPS/WSS \--\> LB

    LB \-- /api/\* \--\> ApiServer  
    LB \-- /ws \--\> WebSocketServer  
    LB \-- /\* \--\> CDN

    CDN \-- 讀取 \--\> WebStorage

    ApiServer \-- 讀寫 \--\> DB  
    ApiServer \-- 寫入 \--\> ImageStorage  
    ApiServer \-- 呼叫 \--\> MapsAPI

    WebSocketServer \-- 讀寫 \--\> DB  
    Driver \-- (即時位置) \--\> WebSocketServer  
    Driver \-- (照片/紀錄) \--\> ApiServer  
    Driver \-- (導航) \--\> MapsAPI
```

## **8\. 工作流程圖 (Workflow Diagrams)**

### **流程 1：管理員規劃路線**

```mermaid
flowchart TD  
    A\[開始\] \--\> B(登入管理員後台)  
    B \--\> C(進入「路線規劃」頁面)  
    C \--\> D(點擊「建立新任務」)  
    D \--\> E(填寫任務日期、名稱，例如: "11/2 路線A")  
    E \--\> F(從「個案列表」拖曳或點選個案)  
    F \--\> G(個案加入「今日路線」列表)  
    G \--\> H{需要調整順序?}  
    H \-- 是 \--\> I(拖曳個案調整順序)  
    H \-- 否 \--\> J(從列表選擇「送餐員」)  
    I \--\> J  
    J \--\> K(點擊「儲存並發佈」)  
    K \--\> L(後端建立 Routes 及 RouteStops)  
    L \--\> M(任務顯示於送餐員 App)  
    M \--\> Z\[結束\]
```


### **流程 2：送餐員送達回報 (含離線)**


```mermaid
flowchart TD  
    A\[開始\] \--\> B(送餐員登入 App)  
    B \--\> C(查看「我的任務」列表)  
    C \--\> D(點擊下一站個案)  
    D \--\> E(點擊「導航」)  
    E \--\> F(抵達地點)  
    F \--\> G(點擊「掃描回報」)  
    G \--\> H(掃描個案 QR Code)  
      
    subgraph "API 驗證"  
        H \--\> I{網路連線?}  
        I \-- 是 \--\> J(API: 驗證 QR Code)  
        J \--\> K{驗證成功?}  
        I \-- 否 \--\> L(本機驗證 QR Code 格式)  
        L \--\> K{驗證成功?}  
    end
```
```mermaid
    K \-- 否 \--\> G(提示: QR Code 錯誤)  
    K \-- 是 \--\> M(自動開啟相機)  
    M \--\> N(強制拍照)  
    N \--\> O(可選: 填寫備註/點選狀態)  
    O \--\> P(點擊「提交完成」)  
      
    subgraph "資料上傳 (含離線)"  
        P \--\> Q{網路連線?}  
        Q \-- 是 \--\> R(上傳照片至 GCS)  
        R \--\> S(上傳紀錄至 API)  
        S \--\> T(App 內標記為「已完成」)  
          
        Q \-- 否 \--\> U(將照片與紀錄暫存於本機)  
        U \--\> V{偵測到網路?}  
        V \-- 否 \--\> U  
        V \-- 是 \--\> W(背景自動上傳資料)  
        W \--\> T  
    end

    T \--\> Z\[結束\]
```
