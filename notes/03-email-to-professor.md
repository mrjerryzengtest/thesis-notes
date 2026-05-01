# 03 — 給教授的郵件草稿

> 狀態：待寄出
> 日期：2026-05-01

---

**主旨：研讀參考資料心得與論文題目初步構想**

---

教授您好，

我已仔細研讀完您上次提供的參考資料，包含 Elastic Sketch（SIGCOMM 2018）、PINT（SIGCOMM 2020）、網路遙測綜述（Minlan Yu, 2019）、Count-Min Sketch 原始文獻，以及相關的技術影片。以下向您簡要回報我的理解與初步構想。

這幾份資料圍繞著同一個核心脈絡：如何在 SDN 網路中進行高效率、低開銷的流量測量，並用於攻擊檢測。Elastic Sketch 提出了一種自適應的 sketch 資料結構，已內建 entropy estimation 功能；PINT 則揭示了一個關鍵限制——機率性子採樣會損害 entropy 估計的準確性——這正好說明了 entropy-based 方法仍有很大的改進空間。

基於以上理解，我初步擬定的論文題目如下：

    SDN 環境下基於多維度熵值分析之網路攻擊精確檢測與分類

    Multi-Dimensional Entropy Analysis for Accurate Network Attack
    Detection and Classification in SDN

主要研究構想是：不依賴單一熵值（如僅計算目的 IP entropy），而是利用 Elastic Sketch 能同時支援多種測量任務的特性，計算多維度熵值向量（如 source IP entropy、destination IP entropy、flow size entropy、port entropy 等），利用不同攻擊類型在熵值空間中的分佈差異，達到更精確的攻擊檢測與分類。此方向也回應了 PINT 論文中提到的 subsampling 對 entropy 的損害問題，可進一步探討如何在低遙測開銷下維持熵值精度。

若您認為這個方向可行，我接下來會先深入研讀 Elastic Sketch 的 technical report，並開始搜尋 entropy-based DDoS detection in SDN 的相關文獻，建立更完整的 related work。

再請教授撥冗給予指導與建議，非常感謝！

祝 研究順利

學生 Jerry 敬上
