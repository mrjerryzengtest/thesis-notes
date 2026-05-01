# 04 — 核心技術洞察

> 從教授提供的參考資料中萃取的關鍵 insight

---

## Insight 1：Entropy Estimation 是 Sketch 的「附加產品」，未經仔細優化

Elastic Sketch 的 entropy estimation 方法非常直接：

```
收集 Light Part 的 counter distribution → 套用 Shannon entropy 公式
```

作者自己也承認這是初步方案（§6.1）：
> "For other tasks (e.g., DDoS, SuperSpreader, and more), we will study how to apply Elastic in the future work."

→ **DDoS 檢測是 Elastic 作者明確提到的 future work！**

→ 這意味著：你的研究方向恰好填補了 Elastic 論文自己指出的空白

---

## Insight 2：CM Sketch 的 Over-estimation Bias 直接污染 Entropy

CM Sketch 的核心特性：
- âᵢ ≥ aᵢ（只有 over-estimation，沒有 under-estimation）
- Error bound：âᵢ ≤ aᵢ + ε‖f‖₁（with probability 1-δ）

這意味著：counter distribution (n₀, n₁, ..., n₂₅₅) **系統性地向右偏移**

對熵值的影響：
- Over-estimated small counters → 實際小 flow 被放大
- 熵值公式 H = -Σ pᵢ log(pᵢ) 在 p 均勻分佈時最大
- Counter bias 使分佈趨向均勻 → **熵值被系統性高估**

→ **Debiasing counter distribution 是提升 entropy 精度的關鍵**

---

## Insight 3：PINT 的悲觀結論 = 你的研究 Motivation

PINT §4.1：
> "Entropy is poorly approximable from subsampled streams."

這句話有兩層含義：
1. **負面**：概率性採樣不適合 entropy estimation（PINT 的自我限制）
2. **正面**：如果有人能解決這個問題，就是一個 significant contribution

→ 你的論文可以正面回應這個挑戰：
> 「即使 PINT 指出 subsampling 損害 entropy，我們證明透過 multi-dimensional entropy
>  和 debiased sketch，仍可達到高精度攻擊檢測」

---

## Insight 4：Elastic 的「通用性」是多維度熵值的天然基礎

Elastic Sketch 的設計目標之一就是 **genericity**：一個資料結構支援六種任務。

這意味著：
- 同一個 Elastic Sketch 實例可以同時提取 source IP entropy、destination IP entropy、port entropy...
- **不需要部署多個 sketch**，避免了資源浪費
- 多維度熵值向量可以從同一個資料結構中**一致地**提取

→ 方案 C（多維度熵值）不是憑空想像，而是 Elastic 設計的自然延伸

---

## Insight 5：攻擊類型在熵值空間中應該是可分的

基於網路攻擊的流量特徵，不同攻擊應該產生不同的熵值指紋：

| 攻擊 | H_srcIP | H_dstIP | H_dstPort | 特徵 |
|------|---------|---------|-----------|------|
| DDoS（偽造源 IP） | ↑↑ | → | → | 高分散來源 |
| Port Scan | → | → | ↑↑ | 高分散埠號 |
| DDoS（Botnet） | ↑ | → | → | 中等分散來源 |
| Normal | → | → | → | 基準線 |

這個假設需要透過實驗驗證，但如果成立，多維度熵值就是一個輕量級的攻擊分類器。

---

## Insight 6：λ 值在攻擊情境下可能需要動態調整

Elastic Sketch 使用固定 λ=8 作為 Ostracism 的驅逐閾值。作者說明 λ ∈ [4, 128] 時精度差異不大。

但在攻擊情境下：
- DDoS 攻擊產生大量小封包 → mouse flow 數量暴增
- 固定 λ 可能導致過多/過少 flow 被歸類為 elephant
- 熵值計算仰賴準確的 elephant/mouse 分離

→ **Attack-adaptive λ 可能顯著提升攻擊期間的 entropy 精度**

---

## Insight 7：Maximum Compression 對 Entropy 的影響是未解問題

Elastic 的 Maximum Compression（MC）能保持 flow size estimation 精度：
> "After Maximum Compression, the error bound is tighter than after Sum Compression."

但 MC 對 entropy estimation 的影響**未被評估**：
- MC 使用 max() 合併 counters → 可能進一步加劇 over-estimation bias
- 在高壓縮比下，counter distribution 可能嚴重失真

→ 這是 entropy-specific 的評估空白，可作為實驗的一部分

---

## Insight 8：已有 P4 Entropy Tracking 的先例

PINT 引用的 Ding et al.（NOMS 2020）：
> "Estimating Logarithmic and Exponential Functions to Track Network Traffic Entropy in P4"

這說明：
1. 在 P4 交換機上做 entropy tracking 是可行的
2. 已有初步工作（使用 log/exp 近似）
3. 你的方法（sketch-based）提供了一條不同的技術路徑

→ 可以將 P4 entropy tracking 作為 related work 中的對比方法

---

## 研究貢獻總結（預期）

1. **理論貢獻**：分析 CM Sketch bias 對 entropy estimation 的影響，推導 debiased error bound
2. **方法貢獻**：提出 multi-dimensional entropy vector 用於攻擊檢測與分類
3. **系統貢獻**：在 SDN 環境中實作並驗證，證明可行性
4. **實驗貢獻**：系統性地比較不同攻擊類型在熵值空間中的行為
