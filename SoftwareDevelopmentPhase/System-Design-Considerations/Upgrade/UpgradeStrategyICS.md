# System Upgrade Strategy | 系統升級策略

![Practical System Design Considerations](../../img/PracticalSystemDesignConsiderations.png)  

## Introduction | 前言   

Software upgrade strategies are often discussed from an Information Technology (IT) perspective, where the primary objectives are service availability, deployment speed, scalability, and user experience. Common examples include cloud services, enterprise resource planning (ERP) systems, customer relationship management (CRM) platforms, e-commerce websites, and Software-as-a-Service (SaaS) applications.   
軟體升級策略通常是以資訊系統（IT System）的角度進行討論，其主要目標包含服務可用性、部署速度、擴充能力以及使用者體驗。常見的應用範圍包括雲端服務、ERP 系統、CRM 系統、電子商務平台以及 SaaS 應用服務。   

However, industrial automation systems operate under fundamentally different constraints. Equipment Front End Modules (EFEM), robot controllers, machine automation systems, motion controllers, and factory equipment management systems interact directly with physical machinery and production processes. In these environments, an unsuccessful upgrade may result not only in software downtime but also in equipment faults, product damage, production interruption, or safety risks.   
然而，工業自動化系統所面對的環境與限制條件截然不同。EFEM（Equipment Front End Module）、機器人控制器、設備自動化系統、運動控制器以及工廠設備管理系統，直接與實體設備及生產流程互動。在這類環境中，一次升級失敗所造成的影響不僅僅是軟體停止服務，還可能導致設備異常、產品損壞、生產中斷，甚至產生安全風險。   

As a result, industrial control systems require a different upgrade strategy from conventional IT systems. The focus shifts from service availability toward operational safety, deterministic behavior, equipment state consistency, and recoverability.   
因此，工業控制系統必須採用與一般 IT 系統不同的升級策略。其核心關注點不再只是服務可用性，而是設備安全性、系統行為可預測性、設備狀態一致性與快速復原能力。   

## Differences Between IT and Industrial Upgrade Strategies | IT 系統與工業控制系統升級策略差異

In a traditional IT environment, the primary objective of an upgrade is to maintain uninterrupted service. Since users continue accessing the system throughout deployment, modern upgrade techniques are designed to keep services online. Techniques such as Blue-Green Deployment, Canary Deployment, and Rolling Upgrade enable new and old software versions to coexist during transition periods. If issues are identified, traffic can simply be redirected back to the previous version.   
在傳統 IT 環境中，升級的首要目標是維持服務不中斷。由於系統升級期間仍有大量使用者持續存取服務，因此現代 IT 系統普遍採用不中斷服務的部署技術，例如 Blue-Green Deployment、Canary Deployment 與 Rolling Upgrade。新舊版本可於轉換期間同時存在，若發現異常，系統可快速將流量切換回舊版本。  

Please refer to the [Online Upgrade Specification for Enterprise IT Systems](Upgrade/OnlineUpgradeIT.md) for details.  
詳細內容請參考 [企業 IT 系統線上升級技術規格書](Upgrade/OnlineUpgradeIT.md)。   

Industrial control systems, however, operate under different assumptions. Physical devices such as robots, conveyors, PLCs, vacuum systems, aligners, elevators, and vision systems may be executing operations when an upgrade is initiated. In these environments, maintaining operational safety is significantly more important than maintaining software availability. Consequently, many industrial systems still prefer controlled offline upgrades for critical control components.   
然而，工業控制系統的運作模式完全不同。當升級開始時，機器手臂、輸送系統、PLC、真空系統、對位裝置或視覺系統可能正在執行設備動作。在此情境下，設備安全遠比服務不中斷更加重要。因此，許多工業控制系統對於核心控制元件仍然傾向採用受控的離線升級方式。   

## Offline Upgrade Strategy | 離線升級策略   

An offline upgrade strategy assumes that the target module is completely stopped before software deployment begins. Database updates, configuration modifications, and user activities are suspended during the upgrade process. This creates a controlled environment in which the state of the system before and after deployment can be precisely compared.   
離線升級策略假設目標模組於升級前完全停止運作。在升級期間，系統不允許資料庫更新、設定檔修改或使用者操作，因此可建立一個受到完整控制的驗證環境。  

Because the data is expected to remain unchanged during the upgrade window, file integrity verification becomes highly effective. Techniques such as manifest comparison, checksum validation, configuration boundary verification, dry-run execution, and health checks are well suited to this model. Any unexpected modification to user data, configuration files, calibration data, or teaching data can be immediately detected and corrected through rollback procedures.   
由於系統資料於升級期間理論上不會發生變化，因此檔案完整性驗證具有很高的可信度。Manifest 比對、Checksum 驗證、設定值邊界檢查、Dry Run 測試以及 Health Check 等技術，都非常適合應用於此類情境。任何未預期的資料變更、Recipe 異動、校正資料改變或設定檔損毀，都能夠立即被偵測並透過 Rollback 流程恢復。   

Offline upgrades prioritize predictability and safety over deployment speed. This approach is commonly adopted for robot motion controllers, PLC communication modules, safety-related services, machine control systems, and semiconductor equipment controllers.  
離線升級的核心目標是確保系統行為可預測且安全可靠，而非追求最快速的部署效率。因此，機器人控制器、PLC 通訊模組、安全控制模組、設備控制核心以及半導體設備控制系統，通常都採用此類策略。   

Please refer to the [Offline Upgrade Validation Specification](OfflineUpgradeICS.md) for details.  
詳細內容請參考 [離線升級驗證技術規格書](OfflineUpgradeICS.md)。   

## Online Upgrade Strategy | 線上升級策略  

Online upgrade strategies are designed to maintain service availability while software updates are being deployed. Since users continue interacting with the system, databases continue processing transactions, and distributed services continue exchanging information, it is unrealistic to compare pre-upgrade and post-upgrade files directly.  
線上升級策略旨在於升級期間維持系統持續提供服務。由於使用者持續操作系統、資料庫持續處理交易，而各項服務之間也持續交換資料，因此無法使用離線升級的思維要求升級前後的檔案完全一致。   

Instead, online upgrade validation focuses on consistency and functionality. Successful deployment is determined by schema compatibility, service health, transaction integrity, API compatibility, communication status, and business process verification. The primary question is not whether the files remain identical, but whether the system continues providing correct functionality throughout and after deployment.  
因此，線上升級的驗證重點並非檔案內容是否相同，而是系統是否維持一致性與正確性。驗證工作主要聚焦於資料庫 Schema 相容性、服務健康狀態、交易完整性、API 相容性、設備通訊狀態以及業務流程驗證。  

Modern IT systems commonly employ Blue-Green Deployment, Canary Deployment, and Rolling Upgrade techniques. These methods reduce deployment risk by gradually introducing new software versions while continuously monitoring system health and user impact.   
現代 IT 系統普遍採用 Blue-Green Deployment、Canary Deployment 與 Rolling Upgrade 等方式進行升級。透過逐步導入新版本並持續監控系統健康狀況，可有效降低部署風險並提高系統可用性。   

Please refer to the [Online Upgrade Validation Specification](OnlineUpgradeICS.md) for details.  
詳細內容請參考 [線上升級驗證技術規格書](OnlineUpgradeICS.md)。   


## Why Manifest Validation Is More Important in OT Systems | 為何 Manifest 驗證更適合 OT 系統

Manifest validation is rarely used as a primary validation mechanism in online IT systems because application data is continuously changing. User activity, transaction records, logs, and operational data naturally evolve over time, making file-level checksum comparisons impractical.  
在一般 IT 系統中，由於資料內容持續變化，因此 Manifest 驗證通常不會成為主要驗證機制。使用者操作、交易紀錄、系統日誌以及各種業務資料都會持續新增或更新，使得檔案層級的 Hash 比對失去實際意義。   

In industrial control systems, however, offline upgrade procedures often freeze all operational activities before deployment. Under these conditions, manifest validation becomes extremely valuable because any data modification occurring during the upgrade process is potentially abnormal and should be investigated.   
然而，在工業控制系統中，離線升級通常會先停止所有設備與系統操作。在此情況下，任何升級期間發生的資料異動都屬於異常現象。因此，Manifest 驗證能夠有效地偵測資料是否遭到非預期修改。   

As a result, manifest comparison is commonly associated with firmware validation, embedded systems, industrial controllers, and equipment management software.   
也因此，Manifest 驗證更常見於韌體升級、嵌入式系統、設備控制器以及工業設備管理軟體。  

## Recommended Hybrid Upgrade Strategy | 建議的混合式升級策略

For modern industrial software platforms, it is often beneficial to adopt a hybrid upgrade strategy rather than relying exclusively on either offline or online deployment methods.   
對於現代工業軟體平台而言，最理想的做法通常不是完全採用 Offline Upgrade 或 Online Upgrade，而是採用混合式升級策略（Hybrid Upgrade Strategy）。   

Critical control components that directly affect machine behavior should continue using offline upgrade procedures. Examples include robot controllers, motion control services, PLC communication modules, safety controllers, and equipment execution engines. These components prioritize safety and deterministic behavior above all other considerations.   
直接影響設備運作的核心控制元件，仍建議使用離線升級。例如機器人控制器、運動控制服務、PLC 通訊模組、安全控制模組以及設備執行引擎等。這些元件應將系統安全性與行為可預測性視為最高優先事項。  

Non-critical services, on the other hand, may benefit from online deployment techniques. Examples include web-based dashboards, monitoring systems, reporting services, analytics platforms, and management portals. These services prioritize availability and user experience and can be upgraded with minimal operational risk.   
至於監控、管理與報表類服務，則較適合採用線上升級方式。例如 Web 管理介面、監控平台、報表服務、資料分析系統與 Dashboard 等。這些元件的重點在於提高可用性與使用者體驗，因此更適合採用 Blue-Green、Canary 或 Rolling Upgrade 等技術。   

## Conclusion | 結論   

The most important distinction between IT and industrial upgrade strategies is their primary objective. IT systems prioritize availability and continuous service, while industrial control systems prioritize safety, predictability, equipment integrity, and recoverability.   
IT 系統與工業控制系統的升級策略最大的差異，在於兩者追求的目標不同。IT 系統重視服務持續可用性，而工業控制系統則更重視設備安全性、系統行為可預測性、設備狀態一致性以及快速復原能力。   

For equipment control platforms such as EFEM systems, robot controllers, and factory automation solutions, a hybrid model is generally the most appropriate architecture. Critical control services should follow offline upgrade procedures with strict validation and rollback mechanisms, while auxiliary services may adopt modern online deployment techniques to improve flexibility and operational efficiency.   
對於 EFEM、機器人控制系統及工廠自動化平台而言，最適合的架構往往是混合式升級模式。設備核心控制服務應採用具備嚴格驗證機制與完整 Rollback 流程的 Offline Upgrade；而監控、管理與報表等周邊服務則可導入現代化 Online Upgrade 技術，以兼顧系統安全性、可用性與維運效率。   

---

# License｜授權條款

![BY NC SA](../../../img/Cc-by-nc-sa.png)   
System Upgrade Strategy © 2026 by Jen Yuan Pan is licensed under the `Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International`.  
系統升級策略 © 2026 作者 潘貞元（Reta Pan），依 `Creative Commons 姓名標示－非商業性－相同方式分享 4.0 國際版` 授權。  

For the complete license terms, rights and obligations, restrictions on use, and other applicable conditions, please refer to the [Full License Terms](https://github.com/retamotome/AgileSoftwareProjectGuide/blob/main/LICENSE.md). By using this work, you acknowledge that you have read, understood, and agree to comply with such terms.  
詳細授權內容、權利義務、使用限制及其他相關條件，請參閱《[完整授權條款](https://github.com/retamotome/AgileSoftwareProjectGuide/blob/main/LICENSE.md)》。您使用本作品，即表示已充分閱讀、確已知悉並同意遵守該等條款與規定，並承認其具有法律約束力。     

---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  
