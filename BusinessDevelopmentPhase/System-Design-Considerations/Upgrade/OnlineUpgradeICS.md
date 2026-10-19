# Online Upgrade Validation Specification | 線上升級驗證技術規格書   

Target System: Industrial Automation Software / EFEM / Robot Controller / Equipment Control System   
適用系統： 工業自動化軟體、EFEM、機器人控制器、設備控制系統   

## Purpose | 目的   

This specification defines the validation requirements and operational procedures for online upgrade deployments. The primary objective is to ensure service continuity, data integrity, system availability, and safe rollback capability while software upgrades are performed without shutting down the entire system.   
本規格書定義系統於線上升級（Online Upgrade）期間之驗證需求與作業程序。  
其主要目的在於確保系統在不完全停機的情況下執行升級作業時，仍能維持：  

* 服務持續運作  
* 資料完整性  
* 系統可用性  
* 快速回復能力  
* 使用者操作不中斷  

Unlike offline upgrades, online upgrades must maintain service availability and support active transactions during deployment.   
與離線升級不同，線上升級期間系統仍可能存在交易、設備通訊及使用者操作，因此必須採用不同的驗證策略。   

## Scope | 適用範圍   

This specification applies to any deployment where the system remains partially or fully operational during the upgrade process.   
本規格適用於升級期間系統仍維持部分或全部服務運作之情境。   

Typical scenarios include:   
典型案例包括：  

* Rolling Upgrade  
* Blue-Green Deployment
* Canary Deployment
* Service-by-Service Upgrade | 微服務逐步升級
* High Availability Architecture Upgrade | 高可用架構（HA）升級  

## Key Difference Between Offline and Online Upgrade | Offline 與 Online 升級差異

During an online upgrade, databases and configuration files may continue to change due to active transactions or ongoing system operations.   
在線上升級模式下，資料庫與設定資料可能持續遭到修改。   
例如：  

* 使用者持續操作系統  
* PLC 持續回報資料  
* Robot 持續執行任務  
* Event Log 持續寫入  

Therefore, validating data through file-level checksums alone is insufficient.   
因此，DB 與 Config 不可能要求升級前後完全相同。  

The validation strategy must focus on consistency, correctness, functionality, and service availability rather than binary equivalence.   
驗證重點應從「檔案是否相同」轉換為：「系統是否維持一致性、正確性與可用性」。   

## Upgrade Architecture Requirements | 升級架構需求  
The system shall support one of the following deployment models:   
系統應支援以下其中一種部署架構：  

**Option A - Rolling Upgrade**  
```
Node A Upgrade | Node A 升級
Node B Running | Node B 持續服務

↓

Node A Running | Node A 恢復服務
Node B Upgrade | Node B 升級
```

**Option B - Blue-Green Deployment**  
```
Blue Environment (Current) | Blue 環境提供服務

↓

Deploy Green Environment | 部署 Green 環境

↓

Switch Traffic | 切換流量

↓

Remove Blue Environment | 下線 Blue 環境
```

**Option C - Canary Deployment**  
```
5% Traffic → New Version | 5% 流量導入新版本

↓

25% Traffic → New Version | 25% 流量導入新版本

↓

100% Traffic → New Version | 100% 流量導入新版本
```

## Pre-Upgrade Validation | 升級前驗證   

Prior to deployment, the system shall verify:   
正式升級前應確認：   

* Backup readiness | 備份機制正常   
* Database replication health | 資料庫同步正常   
* Service health | 服務健康狀態正常   
* Available disk space | 磁碟空間充足   
* Available memory | 記憶體容量充足   
* Cluster status | Cluster 狀態正常
* Network connectivity | 網路連線正常   

## Backup Requirements | 備份需求  

A consistent recovery point shall be created before the upgrade starts.  
升級開始前必須建立一致性復原點（Consistent Recovery Point）。   

The backup shall support point-in-time recovery.   
備份須支援時間點還原（Point-in-Time Recovery）。   

Backup items include:  
備份內容應包含：   

* Database | 資料庫   
* Configuration | 設定資料    
* User Data | 使用者資料   
* Recipe Data
* Calibration Data | 校正資料   
* Software Package | 軟體安裝包   

## Database Validation | 資料庫驗證   

Online upgrades shall validate schema compatibility rather than file consistency.   
線上升級不可採用檔案比對方式驗證資料庫。   

Validation includes:   
驗證重點應包含：   

* Schema version compatibility | Schema Version 相容性   
* Migration success | Migration 成功執行   
* Replication health | 資料同步正常   
* Transaction integrity | 交易完整性維持正常    

**Example**   
Before Upgrade   
```
Schema Version 5
```
After Upgrade   
```
Schema Version 6
Migration Success
No Data Loss
```

## Data Consistency Validation | 資料一致性驗證   

Validation shall verify that user data remains consistent throughout the deployment.   
升級期間必須確認資料維持一致性。   

Examples include:   
驗證方式可包含：   

* Record count validation | 資料筆數驗證   
* Business rule validation | 商業規則驗證   
* Referential integrity validation | 關聯完整性驗證   

**Example**   
```
Order Count Before : 1250
Order Count After  : 1250
```
```
PASS   
```

## Configuration Validation | 設定檔驗證   

Configuration changes shall be validated for compatibility.   
The validation shall include:   
設定驗證應確認：   

* Required parameters | 必要參數存在   
* Data type checking | 資料型別正確   
* Boundary validation | 邊界值合理   
* Version compatibility | 設定版本相容   

## Service Validation | 服務驗證  

Each upgraded service shall be verified independently.   
每個升級完成的服務皆應獨立驗證。  

Validation includes:   
驗證項目包括：  

* Startup success | 啟動成功   
* Health endpoint response | Health Endpoint 正常   
* Dependency connectivity | 相依服務正常   
* Resource consumption | 資源使用正常   

**Example**   
```
Scheduler Service   PASS
Robot Service       PASS
Vision Service      PASS
```

## API Compatibility Validation | API 相容性驗證   

The new version shall remain compatible with existing clients during deployment.   
升級期間新舊版本需能同時運作。   

Validation includes:   
因此必須驗證：  

* REST API compatibility | REST API 相容性   
* gRPC compatibility | gRPC 相容性   
* Message Queue compatibility | Message Queue 相容性   
* Event compatibility | Event 格式相容性   

## Hardware Communication Validation | 設備通訊驗證    

The upgrade shall not interrupt industrial equipment communication.   
升級期間不得影響設備通訊。   

Validation includes:   
驗證項目包括：   

* PLC connection | PLC 連線   
* Robot connection | Robot 連線   
* Camera connection | Camera 連線   
* I/O connection | I/O 通訊   

**Example**  
```
PLC Heartbeat OK
Robot Heartbeat OK
Camera Status OK
```

## Traffic Validation | 流量驗證   

The deployment platform shall validate traffic handling capability.  
升級平台必須持續監控流量狀況。   

Metrics include:  
監控指標包括：   

* Response time | 回應時間   
* Error rate | 錯誤率   
* Throughput | 吞吐量   
* Queue depth | Queue 深度   

**Example**
```
Error Rate < 0.1%
Average Response < 200ms
```

## Canary Validation | Canary 驗證   
For canary deployments, the new version shall be evaluated before wider rollout.   
採用 Canary Deployment 時，必須先驗證新版本表現。   

Validation criteria include:   
驗證項目包括：   

* Crash rate
* Error rate
* Performance impact | 效能影響
* Functional success rate | 功能成功率   

**Example**   
```
Traffic = 5%

Error Rate = 0.01%

PASS
```

## Functional Validation | 功能驗證    

Automated smoke tests shall be executed after deployment.   
部署完成後必須執行 Smoke Test。   

Example workflows:   
例如：

* Login | 使用者登入   
* Job Creation | 建立 Job   
* Recipe Loading | 載入 Recipe   
* Robot Task Execution | 執行 Robot Task   

## Health Check Validation | 健康檢查驗證   

The system shall continuously monitor service health.   
系統必須持續監控健康狀態。   

Verification includes:   
檢查內容包括：   

* Heartbeat
* CPU usage | CPU 使用率   
* Memory usage | 記憶體使用率   
* Database connectivity | 資料庫連線   
* Queue status | Queue 狀態   

## Rollback Requirements | 回復需求   

Rollback shall support zero-data-loss recovery whenever possible.   
回復機制應盡可能達成零資料遺失（Zero Data Loss）。   

Rollback triggers include:   
觸發條件包括：   

* Deployment failure | 部署失敗   
* Schema migration failure | Migration 失敗   
* Health check failure | 健康檢查失敗   
* Service degradation | 服務品質下降   
* Hardware communication failure | 設備通訊異常   

### Rollback Workflow | 回復流程    
```
Failure Detected
偵測到異常
       ↓
Stop Traffic Routing
停止導流至新版本
       ↓
Switch To Previous Version
切換回舊版本
       ↓
Restore Configuration
還原設定資料
       ↓
Restore Database State
還原資料庫狀態
       ↓
Health Check
執行健康檢查
```

## Overall Online Upgrade Sequence | 整體線上升級流程   
```
Phase 0  Health Check | 健康檢查
Phase 1  Create Backup | 建立備份
Phase 2  Deploy New Version | 部署新版本
Phase 3  Database Migration | 執行資料庫 Migration
Phase 4  Service Validation | 服務驗證
Phase 5  API Validation | API 驗證
Phase 6  Hardware Validation | 設備通訊驗證
Phase 7  Canary Validation | Canary 驗證
Phase 8  Traffic Switch | 切換流量
Phase 9  Functional Validation | 功能驗證
Phase 10 Health Check | 健康檢查
Phase 11 Production Release | 正式上線
```

## Acceptance Criteria | 驗收標準   

The online upgrade shall be considered successful only when:   
符合下列條件時，線上升級始可判定為成功：  

* No service interruption occurs during deployment.  
升級期間未發生服務中斷。   
* No data loss is detected.  
未發生資料遺失。  
* Database migration completes successfully.  
資料庫 Migration 成功完成。   
* Hardware communication remains operational.   
設備通訊持續正常。   
* Service health remains within defined thresholds.   
系統健康指標維持於規範範圍內。   
* Functional validation passes.  
功能驗證通過。   
* No rollback condition is triggered.  
未觸發任何回復條件。    

Failure to satisfy any acceptance criterion shall result in an immediate rollback or traffic switchback to the previously validated version. The system shall restore normal service while preserving data consistency and operational continuity.    
若任一驗收標準未通過，系統應立即執行回復程序或流量切換（Traffic Switchback），將服務切換回先前已驗證通過之版本，並確保資料一致性與系統持續可運作。   

---

# License｜授權條款

![BY NC SA](../../../img/Cc-by-nc-sa.png)  
Online Upgrade Validation Specification © 2026 by Jen Yuan Pan is licensed under the `Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International`.  
線上升級驗證技術規格書 © 2026 作者 潘貞元（Reta Pan），採用 `Creative Commons 姓名標示－非商業性－相同方式分享 4.0 國際版` 授權。  

For the complete license terms, rights and obligations, restrictions on use, and other applicable conditions, please refer to the [Full License Terms](https://github.com/retamotome/AgileSoftwareProjectGuide/blob/main/LICENSE.md). By using this work, you acknowledge that you have read, understood, and agree to comply with such terms.  
詳細授權內容、權利義務、使用限制及其他相關條件，請參閱《[完整授權條款](https://github.com/retamotome/AgileSoftwareProjectGuide/blob/main/LICENSE.md)》。使用本文件即表示您已閱讀、理解並同意遵守該等條款。 


---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  
