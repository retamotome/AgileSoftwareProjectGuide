# System Health Monitoring Design Guide｜系統健康監控設計指南

![Practical System Design Considerations](../../img/PracticalSystemDesignConsiderations.png)  

## Key Design Principles｜關鍵設計原則

### **Health ≠ Alive｜健康 ≠ 存活**

* Liveness must include **functional correctness**  
  存活檢測必須包含**功能正確性**

### **Layered Monitoring｜分層監控**

* Combine：  
  整合以下層級：

  * runtime checks  
    執行環境檢查
  * application checks  
    應用層檢查
  * system metrics  
    系統資源指標

### **Fail Fast, Recover Fast｜快速失敗，快速恢復**

* Detect quickly  
  快速偵測問題
* Restart automatically  
  自動執行重啟


### **Event > Polling (when possible)｜優先使用事件驅動（優於輪詢）**

* More scalable  
  更具擴展性
* Lower overhead  
  更低的系統負載

## Native System Health Monitoring｜原生系統健康監控
Please refer to the [Native System Health Monitoring Design Guide](../Native-Healthy-Monitoring.md) for details.    
詳細內容請參考 [原生系統健康監控設計指南](../Native-Healthy-Monitoring.md)。  

## Containerized System Health Monitoring｜容器化系統健康監控
Please refer to the [Containerized System Health Monitoring Design Guide](../Containerized-Healthy-Monitoring.md) for details.    
詳細內容請參考 [容器化系統健康監控設計指南](../Containerized-Healthy-Monitoring.md)。  


## Comparison｜比較

| Aspect<br>面向   | Native System<br>原生系統         | Container System<br>容器化系統                 |
| ---------------- | --------------------- | ------------------- |
| Process control<br>程序控制 | systemd| runtime / orchestrator<br>執行環境 / 編排器            |
| Health detection<br>健康偵測 | internal + watchdog<br>內部監測 + 看門狗   | probe + API<br>探針 + API              |
| Restart<br>重啟機制 | systemd | restart policy<br>重啟策略                  |
| Monitoring<br>監控方式 | netdata / custom<br>netdata / 自訂 | cAdvisor / Prometheus|
| Recovery<br>復原方式 | local restart<br>本地重啟         | distributed / self-healing<br>分散式 / 自我修復            |
| Visibility<br>可視性  | centralized<br>集中式          | distributed → needs aggregation<br>分散式 → 需集中彙整           |

---

# License｜授權條款

![BY NC ND](../../../img/Cc-by-nc-sa.png)  
System Health Monitoring Design Guide © 2026 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
系統健康監控設計指南 © 2026 作者 潘貞元（Reta Pan），依 [姓名-非商業性-相同方式分享 4.0 國際](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  

---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  
