# Practical System Design Considerations | 系統設計實務考量  

![Practical System Design Considerations](../img/PracticalSystemDesignConsiderations.png)  

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

<details>
<summary>News and Reference｜參考資訊</summary>

+ [Fundamentals of Software Architecture: An Engineering Approach](https://www.amazon.com/Fundamentals-Software-Architecture-Comprehensive-Characteristics/dp/1492043451)  
  + [軟體架構原理｜工程方法](https://www.tenlong.com.tw/products/9789865026615)  
+ [Software Architecture Metrics: Case Studies to Improve the Quality of Your Architecture](https://www.amazon.com/Software-Architecture-Metrics-Studies-Improve/dp/1098112237)  
  + [軟體架構指標｜改善架構品質的案例研究](https://www.tenlong.com.tw/products/9786263243583)  

</details>

## Build a Trouble-Shootable System | 建置可排障的系統  

Troubleshooting is never easy, often because "ease of troubleshooting" is rarely considered during the requirement analysis phase. Software engineering is not just about processes, procedures, management, and quality assurance—it also provides mechanisms and methods to build systems that are easier to troubleshoot, especially when issues occur and logs are missing.   
除錯與排障向來不容易，常見原因是「排錯便利性」這類需求在需求分析階段往往未被充分列入考量。軟體工程除了流程、程序、管理與品質保證外，也應提供讓系統在遇到問題或缺乏日誌時仍能較容易排查的機制與作法。  

### Logging Architecture | 日誌架構設計
Please refer to the [Logging Architecture](./Logging-Architecture/README.md) and [Audit Log and System Log](./Logging-Architecture/AuditLog-SystemLog.md) for details.    
詳細內容請參考 [日誌架構設計](./Logging-Architecture/README.md) 以及 [稽核日誌 與 系統日誌](./Logging-Architecture/AuditLog-SystemLog.md)。  


### Resource Usage | 資源使用  

Do **not** assume logs will always be saved when errors occur—especially during critical failures such as crashes, power outages, segmentation faults, or sudden reboots.  
切勿假設在錯誤發生時日誌一定會被保存——尤其是在發生崩潰、斷電、段錯（segfault）或突然重啟等關鍵故障時。  

Fortunately, Linux provides several mechanisms to capture system state (e.g., kernel dumps, memory dumps, process dumps) when a crash or segfault happens and there is no time to write logs.   
幸運的是，Linux 提供多種機制在系統崩潰或段錯且無法即時寫入日誌時擷取系統狀態（例如核心傾印、記憶體傾印、程序傾印等）。  

Always ensure that **at least 30% of system resources** (CPU, memory, disk, etc.) **remain available**, and **implement timely dumps** when sudden failures occur.   
務必確保至少保留 **30% 的系統資源**（CPU、記憶體、磁碟等）可用，並在突發故障時觸發及時的傾印機制。  


> [!note]  
> Data sync operations do **not** guarantee that information is permanently written to disk. Many enterprise‑grade high‑performance drives employ **volatile DRAM in write‑back caching** to accelerate transfers. While this improves throughput, it also introduces risk: if a power failure occurs before cached data is flushed to non‑volatile storage, the data in DRAM is lost, resulting in potential corruption or incomplete writes.  
> 資料同步（Data sync）操作並 **不** 保證資訊會永久寫入磁碟。許多企業級高效能硬碟採用 **具揮發性的 DRAM 寫回快取（write‑back caching）**  來加速傳輸。雖然這能提升效能，但也帶來風險：若在快取資料尚未刷新到非揮發性儲存之前發生斷電，DRAM 中的資料將會遺失，導致潛在的資料毀損或寫入不完整。  
> Please refer to the [Data Processing Strategy](./Data-Processing-Strategy.md) for details.    
> 詳細內容請參考 [資料處理策略](./Data-Processing-Strategy.md)。  


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

### System Health Monitoring｜系統健康監控
Please refer to the [System Health Monitoring](./Healthy-Monitoring/README.md) for details.  
詳細內容請參考 [系統健康監控](./Healthy-Monitoring/README.md)。

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

Please refer to the [Power Failure Handling](./Power-Failure-Handling.md) for details.    
詳細內容請參考 [電力故障處理](./Power-Failure-Handling.md)。

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

# License｜授權條款

![BY NC SA](../../img/Cc-by-nc-sa.png)  
Practical System Design Considerations © 2018 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
系統設計實務考量 © 2018 作者 潘貞元（Reta Pan），依 [姓名-非商業性-相同方式分享 4.0 國際版](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  

---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  
