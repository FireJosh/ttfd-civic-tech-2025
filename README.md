# 臺東縣消防局防災館預約系統

這是一個智慧化的防災館預約與志工媒合平台。

## 系統核心流程

下方是使用者從預約到收到通知的完整互動流程圖。

```mermaid
sequenceDiagram
    autonumber
    
    %% 使用別名 (Alias) 縮短參與者名稱
    actor User as 民眾
    participant FE as 前端網頁
    participant BE as 後端伺erver
    actor Admin as 管理者
    actor Volunteer as 被指派的志工

    %% --- 流程一：民眾提交預約 ---
    User->>FE: 填寫預約表單並點擊送出
    FE->>BE: POST /api/reservations (傳送表單資料)
    Note right of BE: - 儲存案件資料<br>- 狀態設為「待審核」
    BE-->>FE: 回應「申請已提交」
    BE->>Admin: [非同步] 發送 LINE & Email 「新案件通知」

    %% --- 流程二：管理者審核與指派 ---
    Admin->>BE: GET /api/admin/reservations/{id} (查看案件詳情)
    BE-->>Admin: 回傳案件詳情 & 系統自動推薦的5位志工
    
    Admin->>BE: PUT /api/admin/reservations/{id}/approve (送出最終指派的志工名單)
    Note right of BE: - 驗證權限<br>- 更新案件狀態為「已確認」<br>- 綁定指派的志工<br>- 更新該時段為「已額滿」
    BE-->>Admin: 回應「審核成功」

    %% --- 流程三：系統自動化通知 ---
    BE->>User: [非同步] 發送 Email 「預約成功通知」
    BE->>Volunteer: [非同步] 發送 LINE 「新任務通知」
```

## 功能規格

本專案主要分為三大使用者端，各自擁有獨立的核心功能：

### 👨‍👩‍👧‍👦 民眾端功能 (Public-Facing)
- **互動式預約日曆**: 訪客可透過視覺化月曆，清楚查看可預約、已額滿及休館日期。
- **線上預約表單**: 訪客可填寫聯絡資訊、參觀人數、年齡層等，快速完成線上預約。
- **自動化 Email 通知**: 在預約提交後、審核成功後及參訪前，系統會自動寄送通知信件，確保資訊不漏接。

### 🙌 志工端功能 (Volunteer Portal)
- **志工儀表板**: 志工登入後可查看即將到來的導覽任務與個人化資訊。
- **班表管理系統**: 提供視覺化月曆，讓志工能輕鬆點選、更新自己可提供服務的時段。
- **服務時數統計**: 系統會自動累計並顯示服務時數，方便志工隨時查詢。
- **LINE 任務通知**: 當有新的導覽任務被指派時，系統會透過 LINE Bot 即時推播通知給志工。

### ⚙️ 管理員後台功能 (Admin Dashboard)
- **預約案件儀表板**: 集中管理所有預約案件，並可依狀態（待審核、已確認等）進行篩選與查詢。
- **智慧志工媒合**: 系統會根據預約的時段，從班表中自動推薦可服務的志工名單。
- **手動指派與管理**: 管理員保有最終決定權，可手動調整、指派或取消志工的任務。
- **志工資料管理**: 統一管理所有志工的個人檔案與歷史服務紀錄。


# 優化後防災館預約系統 PRD 與流程圖

## 系統架構優化

### 三層式架構設計

```mermaid
graph TB
    subgraph "使用者介面層 / UI Layer"
        PublicUI[民眾預約介面<br/>響應式網頁設計]
        VolunteerUI[志工管理介面<br/>班表與任務管理]
        AdminUI[管理員後台<br/>審核與監控]
    end
    
    subgraph "業務邏輯層 / Business Logic Layer"
        ReservationEngine[預約處理引擎<br/>Reservation Engine]
        MatchingEngine[智慧媒合引擎<br/>Smart Matching Engine]
        NotificationEngine[通知系統引擎<br/>Notification Engine]
        ValidationEngine[資料驗證引擎<br/>Validation Engine]
    end
    
    subgraph "資料存取層 / Data Access Layer"
        Database[(核心資料庫<br/>Core Database)]
        SMSGateway[簡訊閘道<br/>SMS Gateway]
        EmailService[郵件服務<br/>Email Service]
        LineBot[LINE Bot API<br/>訊息推送]
    end
    
    subgraph "監控層 / Monitoring Layer"
        Dashboard[監控儀表板<br/>System Dashboard]
        Logger[日誌系統<br/>Logging System]
        Analytics[數據分析<br/>Analytics Engine]
    end
    
    PublicUI --> ReservationEngine
    VolunteerUI --> ReservationEngine
    AdminUI --> MatchingEngine
    
    ReservationEngine --> ValidationEngine
    ValidationEngine --> Database
    
    MatchingEngine --> Database
    MatchingEngine --> NotificationEngine
    
    NotificationEngine --> SMSGateway
    NotificationEngine --> EmailService
    NotificationEngine --> LineBot
    
    ReservationEngine --> Logger
    MatchingEngine --> Logger
    NotificationEngine --> Logger
    
    Logger --> Dashboard
    Logger --> Analytics
```

## 優化後完整預約流程

```mermaid
sequenceDiagram
    participant U as 民眾
    participant UI as 預約介面
    participant RE as 預約引擎
    participant VE as 驗證引擎
    participant DB as 資料庫
    participant ME as 媒合引擎
    participant NE as 通知引擎
    participant A as 管理員
    participant V as 志工
    participant SMS as 簡訊服務
    participant Email as 郵件服務
    participant LINE as LINE Bot

    %% 民眾預約階段
    U->>UI: 1. 瀏覽可預約時段
    UI->>DB: 查詢可預約日期
    DB-->>UI: 回傳可預約時段清單
    UI-->>U: 顯示互動式日曆
    
    U->>UI: 2. 填寫預約表單
    UI->>VE: 3. 前端資料驗證
    VE-->>UI: 驗證結果
    
    alt 驗證通過
        UI->>RE: 4. 提交預約申請
        RE->>VE: 5. 後端業務邏輯驗證
        VE->>DB: 6. 檢查時段可用性
        
        alt 時段可用
            DB-->>VE: 時段確認可用
            VE-->>RE: 驗證通過
            RE->>DB: 7. 建立預約記錄（狀態：待審核）
            DB-->>RE: 預約ID: #12345
            
            RE->>NE: 8. 觸發確認通知
            NE->>Email: 發送申請確認信
            Email-->>U: 📧 "您的申請已收到（#12345）"
            
            RE-->>UI: 申請成功
            UI-->>U: 顯示成功頁面 + 預約編號
            
        else 時段已滿
            DB-->>VE: 時段已滿
            VE-->>RE: 驗證失敗
            RE-->>UI: 錯誤：時段不可用
            UI-->>U: 請選擇其他時段
        end
        
    else 驗證失敗
        VE-->>UI: 驗證錯誤清單
        UI-->>U: 顯示錯誤訊息
    end

    %% 管理員審核階段
    RE->>A: 9. 通知新預約待審核
    A->>ME: 10. 開啟管理後台審核
    ME->>DB: 11. 查詢符合時段的志工
    DB-->>ME: 回傳可用志工清單
    ME-->>A: 12. 顯示智慧媒合建議
    
    A->>ME: 13. 確認指派志工
    ME->>DB: 14. 更新預約狀態（已確認）+ 指派志工
    
    %% 通知階段
    ME->>NE: 15. 觸發確認通知流程
    
    %% 並行通知所有相關人員
    par 民眾通知
        NE->>Email: 發送預約確認信
        Email-->>U: 📧 預約確認 + 導覽資訊
    and 志工通知
        NE->>LINE: 推送任務通知
        LINE-->>V: 📱 新導覽任務通知
    and 簡訊通知（可選）
        alt 簡訊功能啟用
            NE->>SMS: 發送簡訊提醒
            SMS-->>U: 📱 預約確認簡訊
            SMS-->>V: 📱 任務指派簡訊
        end
    end
    
    %% 錯誤處理與重試
    alt 通知發送失敗
        NE->>NE: 自動重試（最多3次）
        alt 重試失敗
            NE->>A: 通知發送失敗警告
        end
    end
```

## 狀態管理優化流程

```mermaid
stateDiagram-v2
    [*] --> 待審核 : 民眾提交申請
    
    待審核 --> 處理中 : 管理員開始審核
    待審核 --> 已拒絕 : 直接拒絕
    
    處理中 --> 媒合中 : 系統推薦志工
    處理中 --> 已拒絕 : 審核不通過
    
    媒合中 --> 已確認 : 志工指派成功
    媒合中 --> 待審核 : 媒合失敗，重新審核
    
    已確認 --> 進行中 : 導覽當天
    已確認 --> 已取消 : 民眾/管理員取消
    
    進行中 --> 已完成 : 導覽結束
    進行中 --> 異常結束 : 突發狀況
    
    已拒絕 --> [*]
    已取消 --> [*]
    已完成 --> [*]
    異常結束 --> [*]
```

## 品質控制與監控機制

### 自動化驗證標準

```mermaid
flowchart TD
    Input[預約申請輸入] --> Validate{資料驗證}
    
    Validate -->|通過| FormatCheck[格式檢查]
    Validate -->|失敗| ReturnError[返回錯誤訊息]
    
    FormatCheck --> BusinessRule{業務規則檢查}
    BusinessRule -->|通過| ConflictCheck[時段衝突檢查]
    BusinessRule -->|失敗| ReturnError
    
    ConflictCheck -->|無衝突| QualityScore[計算媒合品質分數]
    ConflictCheck -->|有衝突| ReturnError
    
    QualityScore --> ScoreCheck{品質分數 >= 7.0}
    ScoreCheck -->|是| AutoApprove[自動通過]
    ScoreCheck -->|否| ManualReview[人工審核]
    
    AutoApprove --> Success[處理成功]
    ManualReview --> Success
    ReturnError --> End[結束]
    Success --> End
```

### 系統監控指標

| 監控項目 | 目標值 | 警告閾值 | 監控頻率 |
|---------|--------|----------|----------|
| 預約成功率 | ≥ 95% | < 90% | 即時 |
| 媒合成功率 | ≥ 85% | < 80% | 即時 |
| 系統回應時間 | ≤ 2秒 | > 5秒 | 即時 |
| 通知送達率 | ≥ 98% | < 95% | 每小時 |
| 志工參與率 | ≥ 80% | < 70% | 每日 |

## 新增功能規格

### 4. 品質控制模組
- **自動驗證機制**: 預約資料完整性檢查、時段衝突偵測
- **媒合品質評估**: 志工能力與需求匹配度評分（0-10分）
- **異常處理**: 自動重試機制，最多3次重試
- **品質閾值**: 媒合品質分數低於7.0分需人工審核

### 5. 監控與分析模組
- **即時監控儀表板**: 系統狀態、處理量、錯誤率
- **數據分析報表**: 預約趨勢、志工績效、使用者滿意度
- **預警系統**: 異常狀況自動通知管理員
- **效能追蹤**: API回應時間、資料庫查詢效能

### 6. 進階通知系統
- **通知狀態追蹤**: Email/LINE/SMS送達確認
- **個人化通知偏好**: 使用者可自訂接收方式
- **批次通知處理**: 支援大量通知的批次發送
- **通知範本管理**: 可自訂各種通知的內容範本

### 7. 錯誤恢復機制
- **自動重試邏輯**: API失敗時指數退避重試
- **狀態回滾**: 異常情況下自動回滾到上一個穩定狀態
- **資料一致性**: 確保跨系統資料的一致性
- **災難恢復**: 定期備份與快速恢復機制

這個優化版本整合了現代系統設計的最佳實踐，提供更穩定、可擴展的防災館預約系統。
