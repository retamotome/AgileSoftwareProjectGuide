# Offline Upgrade Validation Specification | 離線升級驗證技術規格書  

Target System: Industrial Automation Software / EFEM / Robot Controller / Equipment Control System   
適用系統： 工業自動化軟體、EFEM、機器人控制器、設備控制系統  

## Purpose | 目的   

This specification defines the standard validation process for offline software upgrades and patch deployments. The purpose of the validation process is to ensure that customer data, configuration files, software binaries, runtime dependencies, and hardware communication functions remain intact and operational after the upgrade.   
本規格書定義離線軟體升級（Offline Upgrade）與修補程式（Patch）部署後的標準驗證流程。本驗證流程的目的在於確保系統升級後：

* 客戶資料保持完整  
* 設定檔保持正確  
* 軟體執行檔部署成功  
* 相依元件維持可用  
* 設備通訊功能正常  
* 系統可正常啟動與運作  

The specification also defines backup, rollback, and recovery requirements to minimize system downtime and reduce deployment risks.   
同時規範備份（Backup）、回復（Rollback）及復原（Recovery）機制，以降低停機風險並提升系統可靠度。   

## Scope | 適用範圍   

This specification applies to all offline upgrade activities where the target module or service is stopped prior to deployment.   
本規格適用於所有離線升級作業。升級期間模組停止運作，不允許任何資料存取或設定變更。

The specification covers:  
本規格涵蓋：  

* Database validation | 資料庫驗證  
* Configuration validation | 設定檔驗證  
* Binary validation | 執行檔驗證  
* Dependency validation | 相依元件驗證  
* License validation | 授權驗證  
* Hardware communication validation | 設備連線驗證  
* Functional validation | 功能驗證  
* Rollback validation | 回復驗證   

## Assumptions | 前提條件   

The validation process assumes the following conditions:   
本規格建立於下列前提：   

* Database purge is completed before module shutdown.  
模組停止前已完成 DB Purge。   
* No database transaction exists during upgrade.  
升級期間無任何資料庫交易。   
* No configuration modification occurs during upgrade.  
升級期間無任何設定檔修改。   
* User access is disabled during upgrade.  
升級期間不允許使用者操作。   
* Verification is completed before module restart.  
所有驗證完成後才允許重新啟動模組。   

## Upgrade Workflow | 升級流程   

The standard upgrade workflow shall follow the sequence below:  
標準升級流程如下：    

```
DB Purge
    ↓
Stop Module
停止模組
    ↓
Create Backup Snapshot
建立備份快照
    ↓
Generate Manifest
產生 Manifest
    ↓
Apply Patch
套用 Patch
    ↓
Validation
執行驗證
    ↓
Start Module
啟動模組
    ↓
Health Check
執行健康檢查
```

## Backup Requirements | 備份需求   

A full backup snapshot shall be created prior to patch deployment.  
套用 Patch 前必須建立完整備份快照。   

The backup shall contain:  
備份內容至少應包含：  

* Database files | 資料庫檔案  
* Configuration files | 設定檔  
* Recipe files | Recipe 檔案  
* Calibration data | 校正資料   
* Teaching data  
* License files | 授權檔案   
* Validation manifest | Manifest 檔案  

**Example**  
```
backup/
└─20261016_100000/
    ├─db/
    ├─config/
    ├─recipe/
    ├─calibration/
    ├─license/
    └─manifest.json
```

## Manifest Validation | Manifest 驗證   

A manifest file shall be generated before and after upgrade.  
系統應於升級前後各產生一份 Manifest。  

The manifest shall include:  
Manifest 應記錄：   

* Relative file path | 相對路徑   
* File size | 檔案大小   
* SHA-256 checksum | SHA-256 雜湊值   

**Sample Manifest**  
```
{
  "files": [
    {
      "path": "config/system.yaml",
      "size": 2048,
      "sha256": "A1B2C3..."
    }
  ]
}
```

## Manifest Comparison Rules | Manifest 比對規則   

Validation shall verify:  
系統必須驗證：   

* File count consistency | 檔案數量一致  
* File path consistency | 檔案路徑一致  
* File size consistency | 檔案大小一致  
* SHA-256 consistency | SHA-256 一致  

Failure Conditions  
失敗條件  

* Missing file | 檔案遺失   
* New unexpected file | 發現未預期檔案   
* File content modified | 檔案內容遭修改   
* Checksum mismatch | SHA-256 不一致   

**Example**
```
{
  "result": "FAIL",
  "details": [
    {
      "file": "config/network.yaml",
      "reason": "SHA256_MISMATCH"
    }
  ]
}
```

## Database Validation | 資料庫驗證   

The validation process shall verify:
資料庫驗證應確認：   

* Database file existence | 資料庫檔案存在   
* Database checksum consistency | 資料庫雜湊值一致   
* Schema version compatibility | Schema Version 相容   
* Required tables existence | 必要資料表存在   
* Required indexes existence | 必要 Index 存在   

**Example**   
```
Expected Schema Version : 5
Current Schema Version  : 5
```

## Configuration Validation | 設定檔驗證   

Configuration validation shall include:  
設定檔驗證應包含：   

* Required field validation | 必填欄位驗證   
* Data type validation | 資料型別驗證   
* Boundary validation | 邊界值驗證   
* Configuration version validation | 設定版本驗證   

**Example**  
```
robot_speed: 100
```
Validation rule:   
驗證規則   
```
0 < robot_speed < 1000
```

## Binary Validation | 執行檔驗證

System binaries shall be validated after patch deployment.  
Patch 完成後必須驗證系統執行檔。  

Validation items include:  
驗證項目包括：  

* File existence | 檔案存在性   
* SHA-256 consistency | SHA-256 一致性   
* Software version consistency | 軟體版本一致性   

**Example**  
```
RobotController.exe
Version 3.2.1
```

## Dependency Validation | 相依元件驗證     
All required runtime dependencies shall be verified.   
系統必須驗證所有必要相依元件。   

Examples include:   
例如：  

* .NET Runtime
* Visual C++ Runtime
* OpenSSL
* Python Runtime
* Vendor Drivers | 設備驅動程式   

Verification includes:   
驗證內容包括：  

* Dependency exists | 元件存在    
* Dependency version is correct | 元件版本正確  

## License Validation | 授權驗證   

The upgrade validation shall verify that software licensing remains available after deployment.   
升級驗證必須確認授權機制維持正常運作。   

Verification items include:   
驗證項目包括：   

* License file existence | 授權檔存在   
* License validity | 授權有效   
* License expiration status | 授權未過期   

## Hardware Communication Validation | 設備通訊驗證   

All required equipment connections shall be verified before startup.   
正式啟動前必須驗證所有設備連線狀態。   

Examples include:   
例如：  

* PLC
* Robot Controller | 機器人控制器   
* Camera | 相機系統   
* Barcode Reader | 條碼讀取器   
* I/O Module | I/O 模組   

**Example**    
```
PLC Connected
Robot Connected
Camera Connected
```

## Dry Run Validation | Dry Run 驗證   

A dry run mode shall be executed prior to production startup.   
正式啟動前應執行 Dry Run 模式。   

The dry run shall validate:   
驗證項目應包含：   

* Database loading | 資料庫載入   
* Configuration loading | 設定檔載入   
* Dependency initialization | 相依元件初始化   
* Hardware initialization | 設備初始化   
* License validation | 授權驗證   

**Example**  
```
Database Loading......PASS
Config Loading........PASS
PLC Connecting........PASS
Robot Connecting......PASS
License Check.........PASS
```

## Health Check Validation | 健康檢查驗證   

After startup, the system shall perform health monitoring before proceeding to production operation.   
系統啟動後，必須於投入生產前執行健康檢查。   

Validation items include:   
檢查項目包括：   

* Process running status | 行程執行狀態   
* Heartbeat status | Heartbeat 狀態   
* Communication status | 通訊狀態   
* Memory usage | 記憶體使用量   
* CPU usage | CPU 使用量   
* Error queue status | 錯誤佇列狀態   

## Rollback Requirements | 回復需求   

Any validation failure shall trigger rollback.   
任何驗證失敗都必須觸發回復流程。   

Rollback shall restore:   
回復內容應包含：   

* Database | 資料庫   
* Configuration | 設定檔   
* Recipe
* Calibration data | 校正資料   
* License files | 授權檔   

The previous software version shall be restarted automatically.  
系統應自動重新啟動先前版本。    

### Rollback Workflow | 回復流程

```
Validation Failure
驗證失敗
        ↓
Stop New Version
停止新版本
        ↓
Restore Backup Snapshot
還原備份快照
        ↓
Restore User Data
還原使用者資料
        ↓
Restore Configuration
還原設定檔
        ↓
Start Previous Version
啟動舊版本
        ↓
Health Check
執行健康檢查
```

## Overall Validation Sequence | 整體驗證流程   

```
Phase 0  DB Purge
Phase 1  Stop Module | 停止模組
Phase 2  Backup Snapshot | 建立備份快照
Phase 3  Generate Manifest | 產生 Manifest
Phase 4  Apply Patch | 套用 Patch
Phase 5  Manifest Validation | Manifest 驗證
Phase 6  Database Validation | 資料庫驗證
Phase 7  Configuration Validation | 設定檔驗證
Phase 8  Binary Validation | 執行檔驗證
Phase 9  Dependency Validation | 相依元件驗證
Phase 10 License Validation | 授權驗證
Phase 11 Hardware Validation | 設備通訊驗證
Phase 12 Dry Run Validation | Dry Run 驗證
Phase 13 Start Module | 啟動模組
Phase 14 Health Check | 健康檢查
```

## Acceptance Criteria | 驗收標準   

The upgrade shall be considered successful only when:   
符合下列條件時，升級作業始可判定為成功：   

* All validation phases pass.   
所有驗證階段均通過。  
* No data integrity issues are detected.    
未發現任何資料完整性問題。  
* All required hardware devices are connected.   
所有必要設備皆可正常連線。   
* Health check reports normal status.   
健康檢查結果正常。   
* No rollback condition is triggered.   
未觸發任何回復條件。   

Failure to satisfy any acceptance criterion shall result in an immediate rollback. The system shall restore the previously validated software version, associated user data, configuration files, and runtime environment to ensure operational continuity.   
若任一驗收標準未通過，系統應立即執行回復作業。系統必須還原至先前已驗證通過之軟體版本，並同步還原相關使用者資料、設定檔及執行環境，以確保系統持續維持可運作狀態。  


---

# License｜授權條款

![BY NC SA](../../../img/Cc-by-nc-sa.png)  
Offline Upgrade Validation Specification © 2026 by Jen Yuan Pan is licensed under the `Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International`.  
離線升級驗證技術規格書 © 2026 作者 潘貞元（Reta Pan），採用 `Creative Commons 姓名標示－非商業性－相同方式分享 4.0 國際版` 授權。  

For the complete license terms, rights and obligations, restrictions on use, and other applicable conditions, please refer to the [Full License Terms](https://github.com/retamotome/AgileSoftwareProjectGuide/blob/main/LICENSE.md). By using this work, you acknowledge that you have read, understood, and agree to comply with such terms.  
詳細授權內容、權利義務、使用限制及其他相關條件，請參閱《[完整授權條款](https://github.com/retamotome/AgileSoftwareProjectGuide/blob/main/LICENSE.md)》。您使用本作品，即表示已充分閱讀、確已知悉並同意遵守該等條款與規定，並承認其具有法律約束力。 


---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  
