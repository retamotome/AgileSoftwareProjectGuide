# MCU Diagnostic Framework Design Specification | MCU 診斷框架設計規格書   

**Target Platform**: MCU / RTOS-Based Embedded System  
**Applicable Systems**: EFEM Controller、Robot Controller、PLC Gateway、Equipment Controller、Industrial Automation System   

## Introduction | 前言   

Industrial controllers typically operate under severe resource constraints compared with server or desktop systems. Limited RAM, Flash capacity, storage space, CPU performance, and flash endurance make traditional server-style logging architectures impractical. Excessive trace logging may consume valuable system resources, shorten flash lifetime, and negatively impact real-time control performance.   
相較於伺服器或桌面系統，工業控制器通常受到較嚴格的資源限制。有限的 RAM、Flash 容量、儲存空間、CPU 效能以及 Flash 擦寫壽命，使得傳統伺服器式的 Logging 架構難以直接套用。若大量輸出 Trace Log，不僅會消耗寶貴資源，也可能影響即時控制性能與 Flash 壽命。  

For equipment control systems such as EFEM controllers, robot controllers, PLC gateways, and industrial automation devices, the objective of diagnostics is not to preserve every execution detail. Instead, the primary goal is to retain sufficient operational history to reconstruct system behavior before an abnormal event, allowing efficient fault investigation and recovery.   
對於 EFEM Controller、Robot Controller、PLC Gateway 與各類工業設備控制器而言，診斷系統的目標並非保存所有執行細節，而是在異常發生前後保留足夠的系統事件歷程，協助工程師快速還原問題情境並找出根因。   

This document defines an event-based diagnostic framework that combines Event Ring Buffers, Flash Journaling, Runtime Diagnostic Control, CRC protection, power-failure recovery, and wear-leveling mechanisms to provide a reliable and maintainable diagnostic infrastructure.   
本文件定義一套以 Event 為核心的診斷框架，整合 Event Ring Buffer、Flash Journal、動態診斷控制、CRC 保護、掉電復原以及 Wear Leveling 等機制，以建立高可靠度且適合長期維運的 MCU 診斷系統。   

## Design Objectives | 設計目標   

The framework shall satisfy the following objectives:  
本框架應滿足以下目標：   

* Minimize RAM consumption.  
降低 RAM 使用量。  
* Minimize Flash write operations.  
降低 Flash 寫入頻率。   
* Preserve real-time behavior.  
維持即時控制能力。   
* Support field diagnostics.  
提供現場診斷能力。   
* Support power-loss recovery.  
支援掉電後資料復原。   
* Support long-term Flash endurance.  
延長 Flash 使用壽命。   
* Provide sufficient information for root cause analysis.  
提供足夠資訊進行根因分析。   

## Logging Philosophy | 記錄策略   

This framework adopts an event-oriented approach rather than a trace-oriented approach. Continuous trace messages are useful during development but are generally unsuitable for production systems due to resource constraints and performance impact.   
本框架採用 Event-Oriented Design，而非 Trace-Oriented Design。   
Trace Log 雖然有助於開發階段除錯，但在正式產品中通常會造成過多資源消耗與效能負擔。   

Instead of recording every function call or variable update, the framework records significant operational events such as state transitions, alarms, communication failures, startup sequences, and operator actions.   
因此，本架構不記錄每一次函式呼叫或變數變化，而是記錄具有診斷價值的重要事件，例如狀態切換、設備告警、通訊異常、系統啟動流程以及操作指令等。   

## Trace and Diagnostic Architecture | Trace 與 Diagnostic 架構   

Although TRACE and DIAG (Diagnostic) messages are often grouped together in many software projects, they serve fundamentally different purposes and should be treated as separate diagnostic layers.   
雖然許多軟體專案會將 TRACE 與 DEBUG 視為同一類資訊，但在工業控制系統中，兩者的用途其實完全不同，因此應視為不同層級的診斷資訊。   

TRACE information is intended primarily for software developers during design verification, algorithm validation, performance analysis, and low-level debugging. TRACE messages typically capture execution flow, function entry and exit, state transitions, timing measurements, and internal implementation details.   
TRACE 主要提供軟體開發人員使用，適用於演算法驗證、系統分析、效能分析及程式除錯。TRACE 通常記錄程式執行流程、函式進出、狀態切換、執行時間以及其他內部實作細節。  

Diagnostic information, on the other hand, is intended for field engineers, service personnel, and system integrators. Diagnostic messages provide operational visibility into system behavior without exposing low-level implementation details. They focus on answering operational questions such as system state, equipment status, current mode, active recipe, communication status, and error context.   
Diagnostic（DIAG）則是提供現場工程師、維護人員及系統整合人員使用。DIAG 著重於設備目前的運作狀態，而非程式如何實作。它回答的是設備目前處於什麼狀態、正在執行什麼工作、連線是否正常、是否發生異常等問題。   

For long-life industrial products, TRACE and DIAG should not be considered equivalent and should be managed independently.    
對於生命週期長達十年以上的工業設備產品而言，TRACE 與 DIAG 不應視為相同功能，而應分別管理。   

### TRACE Layer | TRACE 層   

TRACE is a developer-oriented instrumentation layer. Its primary purpose is to reconstruct software execution paths and analyze timing behavior.  
TRACE 屬於開發人員使用的程式追蹤層（Instrumentation Layer），主要用於還原程式執行路徑及分析執行時序。   

Typical TRACE information includes:  
典型 TRACE 內容包含：   

```
Function Entry | 函式進入
Function Exit | 函式離開 
Task Scheduling | Task 排程
State Machine Transition | State Machine 狀態切換
Execution Duration | 執行時間量測
Protocol Parsing Detail | 通訊協定解析過程
```

TRACE data is usually generated at a very high frequency and may significantly affect CPU utilization, memory consumption, communication bandwidth, and storage usage.   
TRACE 通常具有極高的產生頻率，因此可能大幅增加 CPU 使用率、記憶體消耗、通訊頻寬負載以及儲存空間需求。    

For this reason, TRACE should be enabled only in engineering builds and should not be included in production firmware.   
因此建議 TRACE 僅存在於 Engineering Build，不應納入 Release Build。   

**Example | 範例**   

```
Enter MoveTask() 
Send PLC Signal 
Wait PLC ACK 
Received PLC ACK 
Exit MoveTask()
```

### DIAG Layer | DIAG 層  

DIAG is an operational diagnostic layer designed for production systems. Its primary purpose is to provide enough visibility for troubleshooting while maintaining low resource consumption.   
DIAG 屬於產品運行期間的診斷層，其目標是在不消耗過多資源的情況下提供足夠的問題分析能力。   

Unlike TRACE, DIAG should describe system conditions rather than execution details.   
與 TRACE 不同，DIAG 應描述設備目前的運作狀態，而非程式內部執行細節。   

Typical DIAG information includes:   
典型 DIAG 資訊包括：   

```
Current Robot State | 目前 Robot 狀態
Current Recipe | 目前 Recipe
PLC Mode | 目前 PLC 模式
Current Job | 目前 Job
Connection Status | 目前連線狀態
Alarm Context | 異常脈絡資訊
```

Well-designed DIAG information should remain useful even years after product deployment and should be understandable without access to the source code.   
良好的 DIAG 設計即使在產品上市多年後，仍可提供現場工程師獨立分析問題，而不需要查看原始碼。   

**Example | 範例**   

```
Robot State = MOVING 
Current Recipe = PKG_001 
Current Job = LOT12345 
PLC State = READY
```

### Event Layer | Event 層   

Events provide historical context and record significant system activities. Unlike TRACE and DIAG output, events are intended for long-term retention and post-failure analysis.   
Event 層提供事件歷史紀錄與問題追溯能力。不同於 TRACE 與 DIAG，Event 屬於長期保存資料，主要用於故障分析及事件還原。   

Events should be compact, structured, and suitable for storage in RAM buffers or Flash journals.   
Event 應儘量採用結構化格式並降低儲存空間需求。   

**Example | 範例**   

```
MOVE_BEGIN 
MOVE_COMPLETE 
PLC_TIMEOUT 
ALARM_RAISED
```

### Alarm Layer | Alarm 層   

Alarms represent actionable abnormal conditions requiring operator awareness or intervention.   
Alarm 層代表需要立即被操作人員注意或介入處理的異常狀況。   

Alarm records should always be persistent and survive system reboot.   
Alarm 紀錄應永久保存，並於系統重新開機後仍可查閱。   


Examples include:    
例如：   

```
PLC Communication Lost 
Robot Motion Timeout 
Configuration Corrupted 
Emergency Stop Triggered
```

### Recommended Layered Architecture | 建議分層架構   

The recommended diagnostic hierarchy is shown below:   
建議採用以下分層架構：   

```
TRACE
Development Only | 開發追蹤
 
DIAG
Runtime Diagnostic | 執行中診斷
 
EVENT
System History | 系統歷程
 
ALARM
Persistent Failure Record | 異常紀錄
```

Each layer serves a different audience and retention requirement.   
各層級服務不同對象與用途：  

* TRACE targets developers.   
TRACE 提供開發人員進行程式分析。   
* DIAG targets field service engineers.  
DIAG 提供現場工程師進行問題診斷。   
* EVENT targets root-cause investigation.   
EVENT 提供故障追溯與根因分析。   
* ALARM targets operators and maintenance personnel.  
ALARM 提供操作員及維護人員執行異常處置。    


### Recommended Storage Strategy | 建議儲存策略   

For resource-constrained MCU systems, the following storage policy is recommended:   
針對 MCU 平台，建議採用以下儲存策略：   

| Layer<br>層級 | ALARM | Event | DIAG | TRACE |
| ---- | ---- | ---- | ---- | ---- |
| [Severity<br>事件等級](#event-severity-architecture--事件等級架構) | WARNING+ | INFO+ | DIAG | TRACE |
| Storage<br>儲存 | Persistent Flash Storage<br>永久保存於 Flash | RAM Ring Buffer<br>Flash Journal | Runtime Console<br> | Console Only<br>僅輸出 Console<br> | 
| Note<br>備註 | -- | -- | Optional Event Generation<br>必要時轉換為 Event | Engineering Build Only<br>僅限 Engineering Build | 


The proposed layered diagnostic architecture separates development instrumentation, operational diagnostics, historical event recording, and persistent alarm management into independent subsystems. This separation minimizes resource consumption while maintaining sufficient observability throughout the product lifecycle. By limiting TRACE functionality to engineering environments and relying on DIAG, EVENT, and ALARM mechanisms in production environments, the system achieves an optimal balance between diagnosability, reliability, flash endurance, and runtime performance.   
本分層式診斷架構將開發追蹤（TRACE）、運行診斷（DIAG）、事件歷史（EVENT）與異常管理（ALARM）區分為獨立子系統。此設計可在有限的 MCU 資源下兼顧系統可觀測性與執行效能。透過將 TRACE 限制於研發環境使用，而於正式產品中採用 DIAG、EVENT 與 ALARM 機制，可在診斷能力、系統可靠度、Flash 壽命及即時性能之間取得最佳平衡，並提供完整的產品生命週期維護能力。   

## Event Architecture | 事件架構   

Events represent meaningful system activities. Events should describe what happened rather than how the code executed internally.   
Event 是系統內最基本的診斷單位。Event 應描述「發生了什麼事情」，而非程式如何執行。這類事件具有長期保存價值。  

Examples include:   
例如：  

```
Robot Started
Move Completed
PLC Timeout
Alarm Raised
Recipe Loaded
```

### Event Structure | 事件資料結構   

The recommended event structure is shown below.   
建議採用固定長度 Event 結構：  

```
typedef struct
{
    uint32_t timestamp;

    uint16_t module_id;
    uint16_t event_id;

    uint8_t severity;
    uint8_t source;

    uint16_t crc16;

    uint32_t arg1;
    uint32_t arg2;

} event_t;
```

固定長度結構有助於 [Ring Buffer 管理](#ring-buffer-design--ring-buffer-設計)、[Flash 儲存](#flash-journal--flash-journal-設計)與 [CRC 驗證](#crc-protection--crc-保護機制)。


### Dump Format | Dump 格式   

**Human Readable**  
```
Time      : 10:00:01.123
Module    : ROBOT
Event     : MOVE_BEGIN
Severity  : INFO
Arg1      : Slot=5
Arg2      : Port=2
```

**CSV**  
```
timestamp,module,event,severity,arg1,arg2 
1000,ROBOT,MOVE_BEGIN,INFO,5,2 
1200,PLC,TIMEOUT,ERROR,3000,0
```

### Event Dictionary | 事件字典設計   

Events are stored as numeric identifiers rather than text strings.   
系統內部僅儲存 Event ID，不直接儲存文字訊息。   

例如：  

```
EVT_SYS_START = 1000
EVT_PLC_TIMEOUT = 2001
EVT_MOVE_BEGIN = 3000
EVT_ALARM_RAISED = 4000
```

透過 Translation Table 轉換為可讀文字。   

### Event Severity Architecture | 事件等級架構   

Please refer to the [Priority](../Priority.md) for details.    
詳細內容請參考 [重要性權重](../Priority.md) 。   


### Event Source Definition | 事件來源定義  

The source field records the origin of an event.  
Source 用於記錄事件來源。   

```
SRC_ISR
SRC_TASK
SRC_DRIVER
SRC_SYSTEM
```

藉此可快速區分事件來自 ISR、Task 或底層 Driver。

## Runtime Dynamic Log Control | 執行階段動態日誌控制

The framework shall support runtime log configuration without requiring firmware updates or device restarts.   
系統應支援執行期間動態調整日誌等級，而不需要重新燒錄韌體或重新啟動設備。   

Field engineers can dynamically increase diagnostic verbosity during troubleshooting and restore normal operation afterward.   
現場工程師可於故障分析期間提高診斷資訊輸出量，待問題排除後再恢復正常設定。  

### Global Log Control | 全域日誌控制

**Example**:   
範例：   

```
robot_cli log set INFO
```

All modules immediately switch to INFO mode.   
執行後所有模組立即切換至 INFO 等級。   

### Module-Specific Log Control | 模組級別日誌控制   

**Example**:   
範例：   

```
robot_cli log set DIAG robot
```

Only the Robot module enters DIAG (Diagnostic) mode.   
只有 Robot 模組進入 DIAG（暫時性診斷）模式，其餘模組維持原設定。   

### Temporary Diagnostic Session | 暫時性診斷模式   

The framework may support automatic timeout and restoration.   
系統建議支援自動恢復機制。   

**Example**:   
例如：   

```
robot_cli log set DIAG robot 600
```

The setting remains active for 600 seconds before automatically reverting.   
表示 Robot 模組進入 DIAG（暫時性診斷）模式 600 秒，之後自動恢復原設定。   

### Event Recording of Log Changes | 日誌變更事件記錄   

All log-level changes shall themselves be recorded as events.   
所有日誌等級變更都應記錄至 Event Buffer。   

例如：

```
LOG_LEVEL_CHANGED
 
Module = ROBOT
OldLevel = ERROR
NewLevel = DIAG
```

如此可完整追蹤系統診斷行為。   

## Ring Buffer Design | Ring Buffer 設計  

The framework utilizes RAM-based ring buffers to preserve the most recent system history. When the buffer becomes full, the oldest events shall be overwritten.   
系統使用 RAM Ring Buffer 保留最近一段時間的事件紀錄。當 Buffer 滿載時，新事件將覆蓋最舊事件。   

This ensures that the latest events surrounding a failure are always preserved.   
此設計可確保故障發生時能夠保留最近的關鍵事件資訊。   

### ISR and Task Safety Design | ISR 與 Task 安全設計   

Direct sharing of a single ring buffer among multiple ISRs and tasks introduces synchronization complexity and timing uncertainty.   
若 ISR 與多個 Task 共用同一個 Ring Buffer，將增加同步複雜度與即時性風險。  

The recommended architecture separates ISR producers from task producers.   
建議採用分層架構：  

```
ISR Ring Buffer
        ↓
Event Dispatcher
        ↓
Task Ring Buffer
        ↓
Flash Journal
```

This design eliminates contention and minimizes ISR execution time.  
ISR 僅負責快速寫入事件，而後續整合與儲存工作由背景 Task 處理，以降低中斷延遲。  

### Lock-Free Buffer Strategy | 無鎖定 Buffer 策略   

The framework prefers lock-free communication mechanisms over mutex-based synchronization.   
本架構優先採用 Lock-Free 設計，而非 Mutex 或 Semaphore 同步。  

Single Producer Single Consumer (SPSC) ring buffers should be used whenever possible to eliminate locking overhead and reduce real-time jitter.   
若符合 Single Producer Single Consumer 條件，建議直接使用 SPSC Ring Buffer，以降低控制迴圈抖動（Jitter）並提升即時性能。   

## Flash Journal | Flash Journal 設計   

Events requiring long-term retention shall be periodically flushed into Flash Journal storage.   
需要長期保存的事件應由背景 Task 定期寫入 Flash Journal。   

The Flash Journal shall implement append-only behavior.   
Flash Journal 建議採用 Append-Only 模式，以降低 Flash 磨損。   

### Page Structure | Flash 頁面結構   

```
typedef struct
{
    uint32_t magic;
    uint32_t sequence;
    uint32_t timestamp;

    uint16_t event_count;
    uint16_t status;  // WRITING | VALID | BAD

    uint32_t crc32;

} page_header_t;
```

### CRC Protection | CRC 保護機制  

Two layers of CRC protection are recommended.   
Event-level CRC16 protects [individual records](#event-structure--事件資料結構) while page-level CRC32 protects entire storage pages.   

建議採用雙層 CRC 保護機制。  
第一層使用 CRC16 保護[單筆 Event](#event-structure--事件資料結構)。  
第二層使用 CRC32 保護整個 Flash Page。  
如此可同時偵測單筆事件損毀與整頁資料錯誤。   

### Power Loss Recovery | 掉電復原機制   

Event pages shall be committed using a two-phase write procedure.   
系統應採用兩階段提交寫入流程。   

**Step 1**  
```
Status = WRITING
```

**Step 2**  

寫入事件資料。  

**Step 3**  

計算 CRC32。

**Step 4**  
```
Status = VALID
```

Only pages marked VALID shall be considered trustworthy after reboot.   
只有 VALID 頁面可於開機後載入。  

### Boot Recovery | 開機復原流程   

During startup, each page is validated using:  
開機時系統應依序驗證：   

```
Magic
Status
CRC
```

Pages failing any validation check shall be discarded.   
任一項失敗即視為無效頁面並直接忽略。   

### Bad Page Management | 壞頁管理   

Flash pages exhibiting erase failures, write failures, or ECC errors shall be marked `BAD` and removed from future allocation.   
若頁面發生擦除失敗、寫入失敗或 ECC Error，應立即標記為 `BAD`，後續寫入流程不得再次使用該頁面。   

### Wear Leveling Strategy | 磨損分級策略   

Each page maintains an erase counter. The allocator should prefer pages with lower erase counts to distribute wear evenly.   
每個頁面應維護擦寫次數統計。系統應優先使用擦寫次數較低的頁面，以平均消耗 Flash 壽命。  

## Conclusion | 結論  

For MCU-based industrial controllers, a traditional high-volume logging system is neither efficient nor sustainable. An event-driven diagnostic framework combined with lock-free buffering, flash journaling, runtime diagnostic control, CRC protection, power-loss recovery, and wear-leveling provides a significantly more reliable and maintainable solution.   

對 MCU 型工業控制器而言，大量 Trace Log 並非最佳選擇。以 Event 為核心的診斷架構，搭配 Lock-Free Ring Buffer、Flash Journal、Runtime Diagnostic Control、CRC 保護、掉電復原以及 Wear Leveling 機制，能以更低的資源消耗提供更高價值的故障分析能力。  
此架構特別適合 EFEM Controller、Robot Controller、PLC Gateway 與各類工業設備控制系統，可作為產品級診斷框架的設計基礎。  

---

# License｜授權條款

![BY NC SA](../../../img/Cc-by-nc-sa.png)  
MCU Diagnostic Framework Design Specification © 2026 by Jen Yuan Pan is licensed under the `Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International`.  
MCU 診斷框架設計規格書 © 2026 作者 潘貞元（Reta Pan），採用 `Creative Commons 姓名標示－非商業性－相同方式分享 4.0 國際版` 授權。  

For the complete license terms, rights and obligations, restrictions on use, and other applicable conditions, please refer to the [Full License Terms](https://github.com/retamotome/AgileSoftwareProjectGuide/blob/main/LICENSE.md). By using this work, you acknowledge that you have read, understood, and agree to comply with such terms.  
詳細授權內容、權利義務、使用限制及其他相關條件，請參閱《[完整授權條款](https://github.com/retamotome/AgileSoftwareProjectGuide/blob/main/LICENSE.md)》。您使用本作品，即表示已充分閱讀、確已知悉並同意遵守該等條款與規定，並承認其具有法律約束力。     

---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  
