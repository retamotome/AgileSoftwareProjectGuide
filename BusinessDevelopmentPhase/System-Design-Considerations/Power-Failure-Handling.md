# Power Failure Handling | 電力故障處理  

![Practical System Design Considerations](../img/PracticalSystemDesignConsiderations.png)  

## Strategy | 策略

To reduce data loss and system risks during power failures, implement a layered protection strategy.  
為了減少斷電期間的資料遺失與系統風險，應實作分層保護策略。  

- **Power Protection**: Use UPS (Uninterruptible Power Supply) and redundant power supplies to provide buffer time.  
  **電力保護**: 使用 UPS 與冗餘電源供應器以提供緩衝時間。  

- **Power Failure Detection**: Detect voltage drops early using hardware supervisors or ADC (Analog-to-Digital Converter) monitoring, and trigger power-fail interrupts before full power loss occurs.  
  **電力偵測**: 透過硬體監控器或 ADC 提前偵測電壓下降，並在完全斷電前觸發電力異常中斷訊號。  

- **Graceful Shutdown**: Detect power loss and trigger controlled shutdown (flush data, stop services, move hardware to safe state).  
  **優雅關機**: 偵測電力中斷並觸發受控關機（刷新資料、停止服務、將硬體移至安全狀態）。  

- **Data Protection**: Use journaling file systems and ensure critical data is written (fsync/checkpoints), along with storage technologies that support power-loss protection (eMMC/SSD PLP, disable volatile write cache).  
  **資料保護**: 使用日誌式檔案系統並確保關鍵資料已寫入（fsync/檢查點），同時採用支援斷電保護的儲存裝置（如 eMMC/SSD 斷電保護機制，關閉易失性快取）。  

- **Checkpoint & Recovery**: Periodically save system state and support safe restart from last checkpoint.  
  **檢查點與復原**: 定期保存系統狀態並支援從最後檢查點安全重啟。  

- **Transaction Safety**: Use atomic operations and retry mechanisms to prevent partial updates.  
  **交易安全**: 使用原子操作與重試機制以避免部分更新。  

- **Motion & Functional Safety**: Ensure safe-state transition using mechanisms such as Safe Torque Off (STO), controlled stop, and brake engagement where applicable.  
  **運動與功能安全**: 透過安全扭矩關閉（STO）、受控停止與煞車機制確保系統進入安全狀態。  

- **Startup Recovery**: Detect abnormal shutdown and resume or roll back safely.  
  **啟動復原**: 偵測異常關機並安全地恢復或回復先前狀態。  

- **Monitoring & Diagnostics**: Record power-failure events and monitor health of supercapacitors and UPS batteries for predictive maintenance.  
  **監控與診斷**: 記錄電力異常事件並監控超級電容與 UPS 電池健康狀態，以支援預防性維護。  

- **Validation & Testing**: Verify system behavior through repeated power-cut, brownout, and recovery testing to ensure robustness and reliability.  
  **驗證與測試**: 透過反覆斷電、低電壓與復原測試驗證系統行為，以確保穩定性與可靠性。  



## Supercapacitors and UPS | 超級電容與不斷電系統  

In industrial systems, power failure handling is typically achieved through a combination of technologies rather than relying on a single solution. Unlike personal computers, which primarily depend on UPS systems to maintain operation during power loss, industrial equipment adopts a more layered strategy that separates short-term protection from long-term backup needs.   
在工業系統中，電力故障處理通常不是依賴單一方案，而是透過多種技術的組合來實現。不同於個人電腦主要依靠 UPS 來維持斷電時的運作，工業設備會將短期保護與長時間備援分開設計。  

Supercapacitors are commonly used as the first line of defense because they can respond instantly and provide a very short burst of energy, typically lasting from milliseconds to a few seconds. This brief energy supply is sufficient to bridge voltage dips, prevent controller resets, and allow the system to execute a controlled and safe shutdown sequence. In practical design, the required hold-up time must be carefully engineered based on system load (controller, storage, actuators), and the supercapacitor capacity must be sized accordingly.  
超級電容通常作為第一層防護，因為它能夠快速反應並提供極短時間的能量（從毫秒到數秒）。這段時間足以跨越電壓瞬降、防止控制器重啟，並讓系統完成安全關機程序。在實務設計中，需根據系統負載（控制器、儲存裝置、致動器）計算所需撐持時間，並據此進行電容量設定。  

On the other hand, UPS systems are used when a longer backup duration is required. They can supply power for several minutes, enabling systems to continue operating or allowing operators sufficient time to perform manual intervention or orderly shutdown procedures.  
另一方面，當系統需要較長時間的電力支援時，才會使用 UPS。UPS 可提供數分鐘的備援時間，使系統能持續運作或讓操作人員有足夠時間進行人工處理或有序關機。  

For industrial automation systems such as robotic arms or EFEM equipment, the design focus is typically on ensuring a safe transition to a predefined “[Safe Mode](System-Mode-and-State.md)” rather than maintaining continuous operation. When a power failure occurs, the system should stop motion in a controlled manner, preserve essential data such as position or process status, and move actuators to safe positions if necessary. These actions are typically powered by the short hold-up time provided by a supercapacitor, combined with hardware-level safety mechanisms.  
對於機器手臂或 EFEM 等工業自動化系統而言，設計的重點通常是確保設備能安全進入「[安全模式](System-Mode-and-State.md)」，而不是持續運行。在斷電發生時，系統應平穩停止運動、保存關鍵資料（如位置或製程狀態），並在必要時讓致動器回到安全位置，這些動作通常由超級電容的短時間供電及硬體安全機制共同支援完成。  

In practical system design, a supercapacitor is generally considered essential for reliability and protection, while a UPS is added only when continuous operation or extended runtime is explicitly required. This leads to a hybrid architecture in which the supercapacitor handles immediate disturbances and safe shutdown, while the UPS, if present, supports longer operational continuity.  
在實務系統設計中，超級電容通常被視為提升可靠性與保護機制的必要元件，而 UPS 則是在確實需要長時間運作時才導入。因此，常見的架構是以超級電容處理即時電力干擾與安全關機，再視需求加上 UPS 以延長運行時間。   

In summary, industrial systems prioritize “failing safely” over “continuing to run.” Supercapacitors ensure the system can survive a power interruption and protect its state, while UPS systems are used selectively to extend operation when required.   
總結來說，工業系統更重視「安全失效」而非「持續運作」。超級電容的角色是確保系統在斷電時能存活並保護狀態，而 UPS 則是在需要時用來延長運行時間。  

**Comparison | 比較表**

| Solution                | Typical Use                              | Backup Duration         | Purpose                      |
| ----------------------- | ---------------------------------------- | ----------------------- | ---------------------------- |
| **UPS (battery-based)** | PCs, servers, industrial control         | Minutes to hours        | Keep system running          |
| **Supercapacitor**      | Industrial controllers, embedded devices | milliseconds to seconds | Ride-through / safe shutdown |


---

# License｜授權條款

![BY NC SA](../../img/Cc-by-nc-sa.png)  
Power Failure Handling © 2018 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
電力故障處理 © 2018 作者 潘貞元（Reta Pan），依 [姓名-非商業性-相同方式分享 4.0 國際版](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  

---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  
