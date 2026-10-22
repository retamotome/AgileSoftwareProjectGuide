# From Error Handling to Robustness: Understanding Fault Tolerance, Resilience, and Graceful Degradation  
從 Error Handling 到 Robustness：瞭解 Fault Tolerance、Resilience 與 Graceful Degradation   

## Introduction | 前言   

In software engineering, robotics, industrial automation, and distributed systems, reliability is not determined solely by whether a system functions correctly under ideal conditions. More importantly, reliability is determined by how a system behaves when confronted with failures, uncertainties, environmental changes, and unexpected operating conditions. As systems become increasingly complex and interconnected, engineers must design not only for normal operation but also for abnormal situations.   
在軟體工程、機器人、自動化控制與分散式系統領域中，系統的可靠性並不僅取決於其在理想條件下是否能正確運作，更取決於其面對故障、不確定性、環境變化以及各種非預期情境時的表現。隨著系統規模與複雜度不斷提高，工程設計的重點已不再只是滿足正常流程需求，而是如何在異常情境下維持系統的穩定性與可用性。   

The concepts of Error Handling, Fault Tolerance, Resilience, Graceful Degradation, and Robustness represent different aspects of this design philosophy. Although these terms are often used interchangeably in casual discussion, each describes a distinct capability and plays a different role within a system architecture.   
Error Handling、Fault Tolerance、Resilience、Graceful Degradation 與 Robustness 正是描述這些能力的重要概念。雖然在日常討論中人們經常混用這些術語，但從系統工程的角度來看，它們分別代表不同層次的能力，並且在架構設計中扮演不同角色。   

## Error Handling | 錯誤處理    

Error Handling is the foundational mechanism by which a system detects, reports, and responds to errors. Its primary objective is to ensure that when an unexpected condition occurs, the system reacts in a predictable manner rather than failing catastrophically.   
Error Handling 是系統面對異常情況時最基礎的防護機制，其目的在於偵測錯誤、回報錯誤，並採取適當措施處理錯誤。透過良好的錯誤處理機制，系統可以避免因單一異常事件而產生不可預期的崩潰行為。    

In a software application, examples of such errors include invalid input data, unavailable files, network disconnections, database timeouts, and communication failures with external services. The typical response involves logging the error, notifying operators or users, retrying the operation, or terminating the affected process in a controlled manner.    
在軟體系統中，常見的錯誤包括資料格式不符、檔案不存在、資料庫超時、網路連線中斷，以及外部服務無法回應等情況。系統通常會透過記錄日誌、顯示錯誤訊息、重新嘗試執行或安全終止流程等方式來處理這些問題。    

The essential characteristic of Error Handling is that the system is aware that something has gone wrong. However, Error Handling alone does not imply that the system can continue operating. In many cases, successful error handling still results in service interruption.    
Error Handling 的核心特徵在於系統「知道自己出了問題」。然而，它並不保證系統能夠繼續運作。即使錯誤被正確捕捉與處理，服務仍可能因此中斷或停止。    

## Fault Tolerance | 容錯能力    

Fault Tolerance extends beyond merely handling errors. It refers to a system's ability to continue operating correctly despite the failure of one or more components.    
Fault Tolerance 的概念比 Error Handling 更進一步。它描述的是系統在部分元件發生故障時，仍能維持正常運作的能力。   

The fundamental principle behind fault tolerance is the elimination of single points of failure. Rather than assuming all components will remain functional, fault-tolerant systems are designed with redundancy. When a failure occurs, backup components automatically assume responsibility for the failed functionality.    
Fault Tolerance 的核心理念在於消除單點故障（Single Point of Failure）。容錯系統不假設所有元件永遠正常，而是透過備援設計，在主要元件失效時由備援元件自動接手相關工作。   

For example, a database cluster may automatically switch from a failed primary node to a standby replica. Similarly, a robotic system may employ redundant sensors or encoders so that the control loop can continue functioning even when one sensor becomes unavailable.    
例如資料庫叢集中的主節點故障後，由備援節點接手服務；或是在機器人控制系統中使用雙編碼器設計，使控制器在其中一個感測器失效時仍能維持運動控制。    

The defining characteristic of fault tolerance is that service continuity is preserved. Users may not even notice that a failure has occurred because the transition to backup resources happens automatically.    
Fault Tolerance 最重要的特徵在於服務不中斷。理想情況下，故障雖然已經發生，但使用者甚至感受不到系統曾經歷任何異常。   

## Resilience | 韌性   

Whereas Fault Tolerance focuses on preventing interruptions, Resilience focuses on recovery.   
Fault Tolerance 著重於避免服務中斷，而 Resilience 則著重於系統的恢復能力。   

A resilient system acknowledges that some failures are unavoidable and that not all failures can be masked through redundancy. Instead, the system is designed to detect failures, recover from them, rebuild state if necessary, and eventually return to normal operation.    
具有韌性的系統承認故障無法完全避免，也承認並非所有故障都能透過備援機制隱藏起來。因此其設計重點在於故障發生後，系統能夠自動偵測問題、執行恢復流程、重新建立必要狀態，並最終回到正常運作狀態。    

In cloud-native architectures, a container crash may trigger an automatic restart. In industrial automation, a robot collision may require recovery procedures, operator intervention, recalibration, and state reconstruction before production can resume.    
在雲端系統中，容器崩潰後可能由平台自動重新建立執行環境；在工業自動化環境中，機器人撞機後則可能需要經過復歸、重新校正以及狀態同步等流程，才能恢復生產。    

Unlike fault tolerance, resilience allows temporary disruption. Its success criterion is not uninterrupted operation but successful recovery.    
與 Fault Tolerance 不同的是，Resilience 允許短暫中斷。其評估標準並非「是否完全不停機」，而是「是否能夠有效恢復」。    

## Graceful Degradation | 優雅降級    

Graceful Degradation addresses situations where the system cannot maintain full functionality. Rather than failing completely, the system gradually reduces functionality while continuing to provide its most critical services.    
Graceful Degradation 主要處理系統無法維持完整能力的情況。在此情境下，系統並非全面失效，而是選擇逐步降低功能與效能，同時保留最重要的服務能力。   

This design philosophy recognizes that partial service is often preferable to total failure. Consequently, the system selectively disables less critical functions to preserve overall availability.    
這種設計理念認為，即使只能提供部分能力，也通常優於完全停止服務。因此系統會優先保留核心功能，並暫時關閉較不重要的功能，以維持整體可用性。    

A web application may disable recommendation services while preserving search and checkout functionality. A robotic system experiencing sensor degradation may reduce operating speed to maintain safety. An EFEM system with a failed manipulator may continue operating in a lower-throughput configuration using a single robot arm.     
例如電子商務網站可以暫時關閉推薦系統，但仍維持搜尋與結帳功能；機器人在感測器品質下降時，可以主動降低運動速度；EFEM 設備則可能因部分模組故障而切換為單機械手臂模式，以較低產能繼續運作。    

The essential goal of graceful degradation is not to maintain maximum performance, but to avoid complete service loss.    
優雅降級的重點並非維持最高效能，而是在極限條件下避免完全失去服務能力。   

## Robustness | 魯棒性  

Robustness is a broader and more comprehensive concept than the preceding three capabilities.    
相較於前述三種能力，Robustness 是更高階且更全面的概念。   

A robust system continues to function acceptably even when the real world deviates from the assumptions used during design. These deviations may not necessarily be classified as errors or failures. They often originate from uncertainty, variability, or imperfect models.    
魯棒系統即使在現實世界偏離設計假設的情況下，仍能維持可接受的運作效果。這些偏差未必構成程式錯誤或設備故障，而可能來自環境變動、模型誤差或其他不確定因素。    

In robotics, examples include sensor noise, payload variation, component aging, friction changes, mechanical wear, thermal deformation, and environmental disturbances. None of these conditions necessarily produce an exception or hardware fault, yet all can significantly affect system performance.    
在機器人領域中，感測器雜訊、負載變化、機械磨耗、摩擦力改變、熱變形與環境干擾都屬於典型案例。這些問題通常不會直接產生例外訊息，也未必代表硬體故障，但會實際影響控制品質與系統性能。     

A robust design anticipates such uncertainties and incorporates mechanisms that allow the system to maintain stability and acceptable performance despite imperfect information and changing conditions.    
高魯棒性的設計會預先考量這些不確定因素，並透過適當的控制策略與系統架構，使系統即使面臨模型不精確或環境變動，仍能維持穩定且可接受的表現。    

From an engineering perspective, robustness can be viewed as an umbrella concept encompassing error handling, fault tolerance, resilience, and graceful degradation.    
從工程角度來看，Robustness 可以被視為 Error Handling、Fault Tolerance、Resilience 與 Graceful Degradation 的統合能力。    

## Comparative Analysis | 概念比較    

| Concept<br>概念 | Primary Question<br>核心問題 | Main Objective<br>主要目標 | Interruption Allowed<br>是否允許中斷 | Typical Strategy<br>典型策略 |
| --- | --- | --- | --- | --- |
| Error Handling<br>錯誤處理 | What should happen when an error occurs?<br>出錯時如何處理？ | Detect and manage errors<br>偵測與處理錯誤 | Usually Yes<br>通常允許 | Logging, Retry, Exception Handling<br>紀錄、重試、例外處理 |
| Fault Tolerance<br>容錯能力 | Can operation continue despite component failures?<br>故障後能否持續運作？ | Preserve continuity<br>維持服務連續性 | No<br>不允許 | Redundancy, Failover<br>備援、故障切換 |
| Resilience<br>韌性 | Can the system recover after disruption?<br>中斷後能否恢復？ | Restore operation<br>恢復正常運作 | Temporary<br>允許短暫中斷 | Recovery, Rebuild, Restart<br>復原、重建、重啟 |
| Graceful Degradation<br>優雅降級 | Can service continue with reduced capability?<br>能否降級而非崩潰？ | Preserve critical functions<br>保留核心功能 | Depends<br>視情況而定 | Feature Reduction, Performance Limiting<br>功能降級、效能限制 |
| Robustness<br>魯棒性 | Can the system tolerate uncertainty and variation?<br>面對不確定性是否仍可運作？ | Maintain acceptable behavior<br>維持可接受行為 | Depends<br>視設計而定 | Combination of all above approaches<br>綜合運用上述能力 |  


## Conclusion | 結論    

Error Handling, Fault Tolerance, Resilience, and Graceful Degradation should not be viewed as competing concepts. Instead, they represent complementary layers of system reliability.    
Error Handling、Fault Tolerance、Resilience 與 Graceful Degradation 並非彼此競爭的概念，而是構成系統可靠性的不同層次。   

Error Handling ensures that errors are recognized and managed appropriately. Fault Tolerance enables operation to continue despite failures. Resilience provides the ability to recover when failures cause interruptions. Graceful Degradation allows the system to retain critical functions even when operating under degraded conditions. Together, these capabilities contribute to overall robustness.    
Error Handling 確保系統知道如何處理錯誤；Fault Tolerance 確保系統在故障發生時仍能持續運作；Resilience 讓系統在中斷後重新恢復；Graceful Degradation 則讓系統在資源不足或故障加劇時仍保留核心功能。這些能力共同組成了系統的 Robustness。    

A useful engineering perspective is that **Accuracy measures how well a system performs under ideal assumptions, whereas Robustness measures how well it survives in reality**.    
從工程實務角度而言，可以用一句話總結：   
**Accuracy（精度）衡量系統在理想條件下表現有多好；Robustness（魯棒性）衡量系統在現實世界中能活得多好**。   

---

# License｜授權條款

![BY NC SA](../../img/Cc-by-nc-sa.png)  
From Error Handling to Robustness © 2018 by Jen Yuan Pan is licensed under the  `Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International`.  
從 Error Handling 到 Robustness © 2018 作者 潘貞元（Reta Pan），採用 `Creative Commons 姓名標示－非商業性－相同方式分享 4.0 國際版` 授權。  

For the complete license terms, rights and obligations, restrictions on use, and other applicable conditions, please refer to the [Full License Terms](https://github.com/retamotome/AgileSoftwareProjectGuide/blob/main/LICENSE.md). By using this work, you acknowledge that you have read, understood, and agree to comply with such terms.  
詳細授權內容、權利義務、使用限制及其他相關條件，請參閱《[完整授權條款](https://github.com/retamotome/AgileSoftwareProjectGuide/blob/main/LICENSE.md)》。您使用本作品，即表示已充分閱讀、確已知悉並同意遵守該等條款與規定，並承認其具有法律約束力。     

---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  
