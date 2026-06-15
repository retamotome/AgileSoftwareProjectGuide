# Practical System Design Considerations | 系統設計實務考量  

## Overview | 概要  

This section provides a concise overview of key reliability strategies and general considerations essential during the Requirement Analysis phase of software engineering and project management, along with practical tips based on my real-world experience.   
本章節提供在軟體工程與專案管理的需求分析階段中，關鍵可靠性策略與一般考量的簡要概述，並附上我在實務經驗中的一些實用建議。

Companion video: [Lesson 02: Business Requirement Definition](https://github.com/retamotome/SelfDirectedTraining/blob/main/AgileSoftwareEngineering/2025-12-08_BusinessReqDef/README.md).    
參考影片： [Lesson 02: 商業需求定義](https://github.com/retamotome/SelfDirectedTraining/blob/main/AgileSoftwareEngineering/2025-12-08_BusinessReqDef/README.md)。  

For guidance on writing requirements, see [Writing Requirements Effectively](../BusinessRequirementDefinition/Writing-Requirements-Effectively.md).   
關於撰寫需求的實作指引，請參閱 [撰寫需求的高效方法](../BusinessRequirementDefinition/Writing-Requirements-Effectively.md)。 

## System Architecture | 系統架構  

In software engineering, a system is often broadly divided into two parts: User Space (or Application Space) and Kernel Space.   
在軟體工程中，系統通常可大致區分為兩個部份：使用者空間（或應用程式空間）與核心（Kernel）空間。  

When designing in **User Space**, factors like **variability** and **scalability** should be considered from the user's perspective. In **Kernel Space**, **stability** is the top priority, as it is the most critical requirement for the kernel.   
在設計 **使用者空間** 時，應從使用者的觀點考量像是 **多樣性** 與 **可擴充性** 等因素；而在 **核心空間** ，**穩定性** 通常是首要考量，因為它是核心最關鍵的需求。  

There is **no** universal system architecture that fits all scenarios. In reality, **system architecture is influenced by user behavior, hardware constraints, physical environment, contractual limitations, compliance policies, and more**.  
一套放諸四海皆準的系統架構並不存在；實務上系統架構會受到使用者行為、硬體限制、實體環境、合約限制、合規政策等多種因素影響。 


## Build a Trouble-Shootable System | 建置可排障的系統  

Troubleshooting is never easy, often because "ease of troubleshooting" is rarely considered during the requirement analysis phase. Software engineering is not just about processes, procedures, management, and quality assurance—it also provides mechanisms and methods to build systems that are easier to troubleshoot, especially when issues occur and logs are missing.   
除錯與排障向來不容易，常見原因是「排錯便利性」這類需求在需求分析階段往往未被充分列入考量。軟體工程除了流程、程序、管理與品質保證外，也應提供讓系統在遇到問題或缺乏日誌時仍能較容易排查的機制與作法。  


### Audit Log | 稽核日誌  

It is a common misconception that logs are only for developer debugging. In reality,  
**the primary purpose of an audit log is to ensure traceability, accountability, and to help users quickly troubleshoot issues when errors occur**—yes, users, not just developers.  
很多人誤以為紀錄只用於開發者除錯。事實上，  
**稽核日誌的核心目的在於確保可追溯性（traceability）、責任歸屬（accountability），並在錯誤發生時協助使用者快速排障**——是的，協助的是使用者，而不僅僅是開發者。  

When logs are clear and user-friendly, developers can also debug more efficiently, reducing unnecessary escalation and improving overall system usability.  
當日誌清晰且易於理解時，開發者也能更有效率地進行除錯，降低使用者頻繁求助並提升整體系統可用性。  


#### Event Content | 日誌事件內容

An audit log must clearly describe **who did what, when, where, and the result**.  
稽核日誌必須清楚描述「誰在何時於何處進行了什麼操作，以及結果為何」。  
Each log entry should include:  
每一筆日誌應包含以下欄位：  

- **User / Actor ID（使用者或系統行為者）**
- **Action / Event Type（操作或事件類型）**
- **Target / Resource（操作目標或資源）**
- **Result / Status（成功 / 失敗 / 錯誤碼）**
- **Timestamp（時間戳記）**
- **Source（來源，例如 IP / Device ID）**
- **Correlation ID（關聯識別碼，用於跨事件追蹤）**

This structure ensures effective root cause analysis and traceability.

此結構可確保問題追蹤與根因分析能力。


#### Timestamp | 時間戳記  


Accurate timestamps are essential for reliable audit logs. At release, synchronize the system clock using **NTP (Network Time Protocol)** or a **local NTP server** to establish a trusted baseline.  
精確的時間戳記對於可靠的稽核日誌至關重要。發布時應使用 **NTP（網路時間協定）** 或 **本地 NTP 伺服器** 建立可信時間基準。  

For offline systems, maintaining long-term accuracy is challenging. Recommended strategies:  
對於離線系統，長期維持時間準確性是一大挑戰。建議採用以下策略：  

- **Real‑Time Clock (RTC) with battery backup | 具電池備援的實時時鐘（RTC）**  
  - Integrate a hardware RTC chip powered by a coin‑cell battery or supercapacitor.  
	建議整合以鈕扣電池或超級電容供電的硬體 RTC 晶片。  
  - Once initialized via NTP or another trusted source, the RTC continues tracking time independently, even across power cycles.  
	當透過 NTP 或其他可信來源初始化後，RTC 可在斷電或重啟後持續獨立記時。  
  - Accuracy depends on oscillator quality; temperature‑compensated RTCs (TCXO) can limit drift to just a few seconds per year.  
	精確度取決於振盪器品質；採用溫度補償振盪器（TCXO）的 RTC 每年漂移量可降至數秒等級。  
  - Widely regarded as the **standard solution** for offline systems, providing persistent and reliable absolute time.  
    對於離線系統而言，這是被廣泛視為**標準解法**，可提供持久且可靠的絕對時間。  

- **GPS/GNSS receiver | GPS/GNSS 接收器**  
  - Offers highly accurate UTC time directly from satellites.  
	可直接從衛星取得高度準確的 UTC 時間。  
  - Ideal for devices with location hardware or outdoor operation, though it adds cost and complexity.  
	適用於具定位硬體或戶外使用的設備，但會增加成本與系統複雜度。  

- **Local NTP server (during installation) | 安裝時使用的本地 NTP 伺服器**  
  - Useful when internet access is unavailable but a secure LAN is present.  
	當網路不可用但存在可信任的內部網路時很有用。  
  - The device can synchronize with a trusted local server once, ensuring consistent initialization without external connectivity.  
	設備可在安裝時與本地可信伺服器同步一次，確保在沒有外部連線的情況下仍能有一致的初始化時間。  

- **Hybrid logging (absolute + relative uptime) | 混合式日誌（絕對時間 + 相對啟動時間）**  
  - Record both absolute timestamps (from RTC or initial sync) and relative monotonic uptime counters.  
	同時記錄絕對時間戳（來自 RTC 或初始化同步）與相對的單調啟動時間計數器。  
  - This dual approach preserves event order and intervals even if the absolute clock drifts, ensuring audit logs remain reconstructable and trustworthy.  
	此二合一方法能在絕對時鐘出現漂移時仍保留事件順序與時間間隔，確保稽核日誌可重建且具信賴性。 

##### Timestamp Policy｜時間戳記政策
The system time must not be modified, as doing so would lead to inconsistencies in log timestamps and potentially confuse users. This would undermine the primary purpose of maintaining a reliable and trustworthy audit log.  
系統時間不可被修改，因為這將導致日誌時間戳記不一致，並可能使使用者產生混淆，進而違背稽核日誌提供可靠且可追溯紀錄的核心目的。  

If customers require the use of their own timestamps, a separate **“custom timestamp”** field should be introduced rather than overwriting the system time. Customers are permitted to set and update only this custom timestamp.  
若客戶有使用自訂時間的需求，應新增獨立的 **「自訂時間（custom timestamp）」欄位** ，而非覆寫系統時間。客戶僅能設定與更新此自訂時間欄位。  

When the custom timestamp is modified:  
當自訂時間被修改時：  

* The system should calculate and record the **time difference (time delta)** between the system timestamp and the custom timestamp.  
系統應計算並記錄**系統時間與自訂時間之差（time delta）**。  
* Subsequent log entries will continue to use the **system timestamp** as the authoritative reference.  
後續日誌仍以**系統時間**作為唯一且具權威性的基準。  
* The **custom timestamp** will be derived by applying the recorded time difference to the system timestamp.  
**自訂時間**則透過將該時間差套用至系統時間來推導產生。

This approach ensures:  
此設計可確保：  

* The integrity and consistency of system-generated timestamps  
系統時間戳記的完整性與一致性  
* Traceability of any customer-defined time adjustments  
客戶自訂時間調整具備可追溯性  
* Clarity for users when interpreting audit logs  
使用者在解讀稽核日誌時具備清晰性與可理解性   


#### Log Integrity | 日誌完整性

Audit logs must be protected from tampering.  
稽核日誌必須防止竄改。  

Recommended mechanisms:  
建議機制：  

- **Append-only logs（僅允許新增）**
- **Hash chaining（雜湊鏈結）**
- **Digital signatures（數位簽章）**
- **WORM storage（不可修改儲存）**

These ensure logs remain trustworthy for audit and forensic purposes.  
確保日誌在稽核與鑑識分析中具有可信度。  

#### Access Control | 存取控制

Access to audit logs must be strictly controlled.  
日誌存取必須嚴格控管。  

- Only authorized roles can:  
 只有授權的角色才能：   
  - View logs（讀取）
  - Export logs（匯出）  
- No direct modification allowed  
  禁止直接修改  
- Log all access activities  
  存取行為本身也需記錄  

#### Log Retention Policy | 保存政策

Audit logs must define lifecycle management.  
需明確定義日誌生命週期：  

- Retention duration（保存期限，例如 90 天 / 1 年）
- Log rotation（輪替機制）
- Archiving（歸檔策略）
- Deletion policy（刪除需可追蹤）

#### Severity & Readability | 嚴重性與可讀性

To improve usability:  
為提升可用性：  

- Define severity levels:  
  定義嚴重等級：
  - INFO / WARNING / ERROR / CRITICAL  
    訊息 / 警告 / 錯誤 / 嚴重  
- Use human-readable messages  
  使用易懂文字  
- Avoid unnecessary technical jargon  
  避免過度技術術語  
- Provide actionable hints when possible  
  盡可能提供解決建議  

#### Log Export & Integration | 匯出與整合

Support integration with external systems:  
支援與外部系統整合：  

- Export formats（檔案 / USB / API / Syslog）
- Optional:
  - Central log server（集中式日誌伺服器）
  - SIEM integration（資安事件管理系統）

#### Design Principles | 設計原則

A well-designed audit log should be:  
良好的稽核日誌應具備：  

- **Reliable（可靠）**
- **Tamper-resistant（防竄改）**
- **Traceable（可追溯）**
- **User-friendly（易於理解）**
- **Consistent（一致性）**


### Resource Usage | 資源使用  

Do **not** assume logs will always be saved when errors occur—especially during critical failures such as crashes, power outages, segmentation faults, or sudden reboots.  
切勿假設在錯誤發生時日誌一定會被保存——尤其是在發生崩潰、斷電、段錯（segfault）或突然重啟等關鍵故障時。  

Fortunately, Linux provides several mechanisms to capture system state (e.g., kernel dumps, memory dumps, process dumps) when a crash or segfault happens and there is no time to write logs.   
幸運的是，Linux 提供多種機制在系統崩潰或段錯且無法即時寫入日誌時擷取系統狀態（例如核心傾印、記憶體傾印、程序傾印等）。  

Always ensure that **at least 30% of system resources** (CPU, memory, disk, etc.) **remain available**, and **implement timely dumps** when sudden failures occur.   
務必確保至少保留 **30% 的系統資源**（CPU、記憶體、磁碟等）可用，並在突發故障時觸發及時的傾印機制。  


> [!note]  
> Data sync operations do **not** guarantee that information is permanently written to disk. Many enterprise‑grade high‑performance drives employ **volatile DRAM in write‑back caching** to accelerate transfers. While this improves throughput, it also introduces risk: if a power failure occurs before cached data is flushed to non‑volatile storage, the data in DRAM is lost, resulting in potential corruption or incomplete writes.  
> 資料同步（Data sync）操作並 **不** 保證資訊會永久寫入磁碟。許多企業級高效能硬碟採用 **具揮發性的 DRAM 寫回快取（write‑back caching）** 來加速傳輸。雖然這能提升效能，但也帶來風險：若在快取資料尚未刷新到非揮發性儲存之前發生斷電，DRAM 中的資料將會遺失，導致潛在的資料毀損或寫入不完整。  


### Memory Dump Strategy | 記憶體傾印策略

- Perform memory dumps **after processing large datasets in memory**, particularly when the system is under heavy load or showing performance degradation.  
  在記憶體處理大量資料集或系統負載高、性能下降時，執行記憶體傾印。  
- Dumps provide a **snapshot of the system state**, helping prevent data loss and enabling detailed post‑mortem analysis.  
  傾印提供系統狀態的快照，有助於防止資料遺失並支援詳細的事後分析。  
- They are essential for diagnosing **memory leaks, bottlenecks, and hidden issues** that may not be visible through logs alone.  
  傾印對於診斷 **記憶體洩漏、效能瓶頸與隱藏問題**（單靠日誌可能無法察覺）非常重要。  
- Regularly scheduled or event‑triggered dumps improve **system reliability and troubleshooting efficiency**.  
  定期排程或事件觸發的傾印能提升系統可靠度與排障效率。  
- Ensure dumps are securely stored and accessible for analysis, while considering **data sensitivity and compliance requirements**.  
  確保傾印檔案以安全方式儲存且可供分析，同時顧及資料敏感性與合規性要求。  

## Build a Quality-Assurable System | 建立可品質保證的系統   

### System Mode and State | 系統模式與狀態   
Please refer to the [System Mode and State](./System-Mode-and-State.md) for details.  
詳細內容請參考 [系統模式與狀態](./System-Mode-and-State.md)。

## Failure Tolerance Strategy | 故障容忍策略  

### Redundancy and Failover | 冗餘與故障切換  

Provide **high availability** and **service continuity** by ensuring critical components can automatically recover or switch to backup systems when failures occur.  
透過確保關鍵元件能在故障時自動復原或切換至備援系統，以提供 **高可用性** 與 **服務持續性**。  

* Deploy **hot standby / failover systems** for critical services  
  為關鍵服務部署 **熱備援 / 故障切換系統**  
* Use **replication mechanisms** (sync or async based on RPO requirements)  
  使用 **複製機制**（依據 RPO 要求選擇同步或非同步）  
* Ensure failover nodes have **independent power sources**  
  確保故障切換節點具備 **獨立電源來源**  


### Packaging Modules in Docker Images | 將模組封裝於 Docker 映像檔  

Most systems do not implement full redundancy due to cost and complexity. Instead, a **containerized architecture** is used, where each module runs as an independent Docker container.  
多數系統因成本與複雜度未實作完整冗餘，而是採用 **容器化架構**，讓每個模組以獨立的 Docker 容器執行。  

* Failures are **isolated per module**  
  故障會被 **隔離在單一模組**  
* Only the affected container needs to be **restarted**  
  僅需 **重新啟動受影響的容器**  


### A/B Update Mechanism | A/B 更新機制  

To ensure safe updates and prevent system corruption, the A/B Update strategy uses two identical volumes managed by an Update Manager.  
為了確保更新安全並避免系統損壞，A/B 更新機制使用由更新管理器控制的兩個相同磁碟區。  

* If booting from volume B fails, the system automatically reverts to volume A, erases volume B, and restores it from volume A.  
  若從磁碟區 B 開機失敗，系統會自動回復至磁碟區 A，清除磁碟區 B，並由 A 還原。  
* If booting from volume B succeeds, volume A is erased and synchronized with the contents of volume B.  
  若從磁碟區 B 開機成功，磁碟區 A 會被清除並與 B 的內容同步。  

### Power Failure Handling | 電力故障處理  

To reduce data loss and system risks during power failures, implement a layered protection strategy.  
為了減少斷電期間的資料遺失與系統風險，應實作分層保護策略。  

- **Power Protection**: Use UPS and redundant power supplies to provide buffer time.  
  **電力保護**: 使用 UPS 與冗餘電源供應器以提供緩衝時間。  

- **Graceful Shutdown**: Detect power loss and trigger controlled shutdown (flush data, stop services, move hardware to safe state).  
  **優雅關機**: 偵測電力中斷並觸發受控關機（刷新資料、停止服務、將硬體移至安全狀態）。  

- **Data Protection**: Use journaling file systems and ensure critical data is written (fsync/checkpoints).  
  **資料保護**: 使用日誌檔檔案系統並確保關鍵資料已寫入（fsync/檢查點）。  

- **Checkpoint & Recovery**: Periodically save system state and support safe restart from last checkpoint.  
  **檢查點與復原**: 定期保存系統狀態並支援從最後檢查點安全重啟。  

- **Transaction Safety**: Use atomic operations and retry mechanisms to prevent partial updates.  
  **交易安全**: 使用原子操作與重試機制以避免部分更新。  

- **Startup Recovery**: Detect abnormal shutdown and resume or roll back safely.  
  **啟動復原**: 偵測異常關機並安全地恢復或回復先前狀態。  


## Load Balancing Strategy | 負載平衡策略  

### CPU Affinity | CPU 親和性  

To prevent heavy processes from overwhelming shared CPU resources and starving other critical services, **CPU affinity should be carefully designed and enforced**.  
為了避免高負載程序壓垮共享 CPU 資源並使其他關鍵服務資源不足，必須 **謹慎設計並強制執行 CPU 親和性**。  

* Assign CPU cores explicitly to high-load or real-time processes to ensure predictable performance.  
  明確分配 CPU 核心給高負載或即時程序，以確保可預測的效能。  
* Isolate critical services from non-critical workloads to avoid resource contention.  
  將關鍵服務與非關鍵工作負載隔離，以避免資源競爭。  
* Avoid unrestricted process scheduling, which may lead to CPU saturation, increased latency, or system stalls.  
  避免不受限制的程序排程，因為這可能導致 CPU 飽和、延遲增加或系統停滯。  
* Consider NUMA awareness (if applicable) to further optimize performance on multi-socket systems.  
  在多插槽系統中，若適用，應考慮 NUMA 感知以進一步最佳化效能。  

Proper CPU affinity planning helps achieve **deterministic execution**, improved system stability, and better overall resource utilization.  
良好的 CPU 親和性規劃有助於達成 **確定性執行**、提升系統穩定性並改善整體資源利用率。  

### NIC Binding (Network Interface Bonding / Teaming) | 網路介面綁定  

NIC binding combines multiple network interfaces to improve both **bandwidth availability** and **fault tolerance**.  
網路介面綁定能結合多個網路介面，以提升 **頻寬可用性** 與 **故障容忍度**。  

* **Increased bandwidth**: Traffic can be distributed across multiple NICs, reducing bottlenecks.  
  **增加頻寬**：流量可分散至多個網路介面，降低瓶頸。  
* **High availability**: If a single network interface fails, traffic is automatically redirected to remaining interfaces.  
  **高可用性**：若某一網路介面故障，流量會自動導向其他介面。  
* **Improved reliability**: Reduces the risk of connection loss due to single point of failure.  
  **提升可靠性**：降低因單點故障導致連線中斷的風險。  

Depending on the system design, NIC binding can be configured in modes such as:  
依系統設計不同，網路介面綁定可設定為以下模式：  

* Active-backup (failover-focused)  
  主備模式（以故障切換為主）  
* Load balancing (throughput-focused)  
  負載平衡模式（以吞吐量為主）  
* LACP (Link Aggregation Control Protocol, IEEE 802.3ad) for dynamic link aggregation  
  LACP（鏈路聚合控制協定，IEEE 802.3ad）動態鏈路聚合  

This ensures **resilient and scalable network performance**, especially in high-throughput or mission-critical environments.  
這能確保 **具備彈性且可擴展的網路效能**，特別適用於高吞吐量或任務關鍵的環境。  


## Data Processing Strategy | 資料處理策略    
Please refer to the [Data Processing Strategy](./Data-Processing-Strategy.md) for details.  
詳細內容請參考 [資料處理策略](./Data-Processing-Strategy.md)。

## Security Consideration | 安全性考量   
Please refer to the [Security Consideration](./Security-Consideration.md) for details.    
詳細內容請參考 [安全性考量](./Security-Consideration.md)。

---

## License｜授權條款

![BY NC ND](../../img/Cc-by-nc-sa.png)  
Practical System Design Considerations © 2018 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
系統設計實務考量 © 2018 作者 潘貞元（Reta Pan），依 [姓名-非商業性-相同方式分享 4.0 國際](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  

---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  
