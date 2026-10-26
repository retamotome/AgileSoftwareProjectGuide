# Reta's Agile Software Project Guide｜Reta 的敏捷軟體專案指南  

![Agile Software Project Guide](./img/AgileSoftwareProjectGuide.png)

## Overview | 概要  

A concise, phase-based guide for planning and executing Agile software projects. This repository documents each project phase, recommended practices, and links to supporting videos and artifacts.  
本指南提供以「階段」為主軸的精簡指南，說明如何規劃與執行敏捷軟體專案。本儲存庫涵蓋各專案階段、建議實務，以及相關教學影片與產出文件連結。

## Learning Objectives｜學習目標

* Identify the primary phases of an Agile project and their objectives.  
  辨識敏捷專案的主要階段及其目標。

* Apply practical techniques to improve code quality, testing, and delivery.  
  應用實務技巧，提升程式碼品質、測試品質與交付效率。

* Strengthen team collaboration and Agile processes.  
  強化團隊協作與敏捷流程運作。

## Video｜教學影片  

| Video 影片 | Description 說明 | 
|:-----|:-------|
| <a href="https://github.com/retamotome/SelfDirectedTraining" target="_blank"><image src="https://github.com/retamotome/SelfDirectedTraining/raw/main/img/SelfDirectedTraining.png" width="200"></a> | [Reta's Self-Directed Learning Guide｜Reta 的自主學習指南](https://github.com/retamotome/SelfDirectedTraining) <br>This repository contains resources and guidance for self-directed learning, focusing on practical applications and real-world scenarios.<br>本儲存庫包含自主學習的資源與指南，專注於實務應用與現實情境。<br> ☑️ English ☑️ Chinese  |

## Tutorial Overview｜指南總覽

This guide covers practical activities in software projects and software engineering across initiation, development, testing, and maintenance.  
本指南涵蓋軟體專案以及軟體工程在啟動、開發、測試與維護各階段的實務活動。

For an introductory presentation, see:  
入門介紹請參考：  
[Lesson 01: Agile and Software Engineering｜敏捷與軟體工程全覽](https://github.com/retamotome/SelfDirectedTraining/blob/main/AgileSoftwareEngineering/2025-11-09_AgileProjectMgt/README.md)  

## Key Project Phases｜專案主要階段

### Project Initiation｜專案啟動階段

* Project planning, team formation, and environment setup.  
  專案規劃、團隊組成與開發環境建置。

* Objective: define the project vision, roles, and a high-level roadmap.  
  目標：定義專案願景、角色分工及高階發展藍圖。

| Guidance 指南 | Description 說明 | 
|:-----|:-------|
| <a href="./ProjectInitiationPhase/README.md" target="_blank"><image src="./img/ProjectInitiationPhase.png" width="200"></a> | [Project Initiation Phase｜專案啟動階段](./ProjectInitiationPhase/README.md) <br>TThe initiation phase establishes the foundation for a successful Agile software project. Key activities include defining the project vision, assembling the team, and setting up initial plans and practices.<br>啟動階段為成功的敏捷軟體專案奠定基礎。主要活動包括定義專案願景、組建團隊，以及建立初始計畫與實務運作方式。 |

### Business Development｜商業(需求)發展階段  

* Market research, business case development, and stakeholder engagement.  
  市場調查、商業案例建立與利害關係人溝通。

* Building on [Lesson 02: Business Requirement Definition](https://github.com/retamotome/SelfDirectedTraining/blob/main/AgileSoftwareEngineering/2025-12-08_BusinessReqDef/README.md) to **capture and validate stakeholder needs**.  
  依照 [Lesson 02: 商業需求定義](https://github.com/retamotome/SelfDirectedTraining/blob/main/AgileSoftwareEngineering/2025-12-08_BusinessReqDef/README.md) 之流程，**蒐集與驗證利害關係人需求**。

  * Systematically elicit, document, and confirm stakeholder requirements to ensure alignment with business objectives and stakeholder expectations.  
    以系統化方式蒐集、記錄並確認需求，確保與商業目標及利害關係人期望一致。

* This phase intentionally **excludes implementation-level technical details**.  
  此階段刻意**不包含實作層級的技術細節**。

| Guidance 指南 | Description 說明 | 
|:-----|:-------|
| <a href="./BusinessDevelopmentPhase/README.md" target="_blank"><image src="./img/BusinessDevelopmentPhase.png" width="200"></a> | [Business Development Phase｜商業(需求)發展階段](./BusinessDevelopmentPhase/README.md) <br>This phase focuses on defining clear, testable business requirements and preparing a foundation for behavior-driven validation.<br>本階段著重於定義清楚且可測試的商業需求，並建立行為驅動驗證（Behavior-Driven Validation）的基礎。 |

### Software Development｜軟體發展階段

* Iterative design, implementation, testing, and integration to deliver working software.  
  透過反覆迭代進行設計、實作、測試與整合，交付可運作的軟體。

| Guidance 指南 | Description 說明 | 
|:-----|:-------|
| <a href="./BusinessDevelopmentPhase/System-Design-Considerations/README.md" target="_blank"><image src="./BusinessDevelopmentPhase/img/PracticalSystemDesignConsiderations.png" width="200"></a> | [Practical System Design Considerations｜系統設計實務考量](./BusinessDevelopmentPhase/System-Design-Considerations/README.md) <br>This section provides a concise overview of key reliability strategies and general considerations essential during the Requirement Analysis phase of software engineering and project management, along with practical tips based on my real-world experience.<br>本章節提供在軟體工程與專案管理的需求分析階段中，關鍵可靠性策略與一般考量的簡要概述，並附上我在實務經驗中的一些實用建議。 |

| Video 影片 | Description 說明 | 
|:-----|:-------|
| <a href="https://github.com/retamotome/retamotome/blob/main/open-courseware/SystemDesign.md" target="_blank"><image src="https://github.com/retamotome/retamotome/raw/main/open-courseware/img/SystemDesignPhilosophy.png" width="200"></a> | [以四大關鍵思維看系統設計哲學](https://github.com/retamotome/retamotome/blob/main/open-courseware/SystemDesign.md)<br>本課程結合系統思維、軟體工程原則與跨領域觀點，探索系統設計的核心。透過挑戰既有假設、分析取捨，培養學習者洞察問題根源、化繁為簡，並尋找能夠優化整體系統的有效解決方案之能力。<br>This course explores system design through systems thinking, software engineering principles, and interdisciplinary perspectives. By challenging conventional assumptions, analyzing trade-offs, learners will develop the ability to identify root causes, simplify complexity, and discover effective solutions that optimize the system as a whole. <br> ☑️ Chinese  |  

### Maintenance｜維護階段

* Post-deployment bug fixes, performance improvements, and feature enhancements.  
  上線後之錯誤修復、效能最佳化與功能擴充。



## About Test Engineering｜關於測試工程

> [!note]  
> Both the **Business Development** and **Software Development** phases include practices on:  
> **商業需求發展**與**軟體發展**階段皆包含以下實務：  
> * Software engineering practices 軟體工程實務。  
> * Test engineering practices 測試工程實務。  

> [!important]   
> Test engineering in **Business Development** focuses on validating **business requirements** using **behavior-driven techniques (BDD)**.   
> 在**商業(需求)發展階段**，測試工程重點為使用**行為驅動開發（BDD）** 來驗證**商業需求**。  
>  
> Test engineering in **Software Development** focuses on verifying **implementation correctness** using **test-driven approaches (TDD)**.  
> 在**軟體開發階段**，測試工程重點為使用**測試驅動開發（TDD）** 來驗證**實作正確性**。  

---

# License｜授權條款

![BY NC SA](./img/Cc-by-nc-sa.png)     
This work © 2026 by Jen Yuan Pan is licensed under the `Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International`.  
本作品 © 2026 作者 潘貞元（Reta Pan），採用 `Creative Commons 姓名標示－非商業性－相同方式分享 4.0 國際版` 授權。  

For the complete license terms, rights and obligations, restrictions on use, and other applicable conditions, please refer to the [Full License Terms](https://github.com/retamotome/AgileSoftwareProjectGuide/blob/main/LICENSE.md). By using this work, you acknowledge that you have read, understood, and agree to comply with such terms.  
詳細授權內容、權利義務、使用限制及其他相關條件，請參閱《[完整授權條款](https://github.com/retamotome/AgileSoftwareProjectGuide/blob/main/LICENSE.md)》。您使用本作品，即表示已充分閱讀、確已知悉並同意遵守該等條款與規定，並承認其具有法律約束力。     

---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  

