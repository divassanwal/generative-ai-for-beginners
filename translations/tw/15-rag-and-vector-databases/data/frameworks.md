<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "b5466bcedc3c75aa35476270362f626a",
  "translation_date": "2025-07-09T16:27:27+00:00",
  "source_file": "15-rag-and-vector-databases/data/frameworks.md",
  "language_code": "tw"
}
-->
# 神經網路框架

如我們之前所學，要有效訓練神經網路，我們需要做到兩件事：

* 操作張量，例如乘法、加法，以及計算一些函數如 sigmoid 或 softmax
* 計算所有表達式的梯度，以便進行梯度下降優化

雖然 `numpy` 函式庫可以完成第一部分，但我們還需要某種機制來計算梯度。在前一節我們開發的框架中，必須手動在 `backward` 方法中編寫所有導數函數，該方法負責反向傳播。理想的框架應該能讓我們計算*任何定義的表達式*的梯度。

另一個重要的點是能夠在 GPU 或其他專用計算單元（如 TPU）上執行運算。深度神經網路訓練需要*大量*計算，能在 GPU 上並行化這些計算非常關鍵。

> ✅ 「並行化」指的是將計算分配到多個裝置上同時進行。

目前最受歡迎的兩大神經網路框架是 TensorFlow 和 PyTorch。兩者都提供低階 API，可在 CPU 和 GPU 上操作張量。在低階 API 之上，分別有高階 API，稱為 Keras 和 PyTorch Lightning。

低階 API | TensorFlow | PyTorch
---------|-------------|---------
高階 API | Keras       | PyTorch Lightning

兩個框架的**低階 API**允許你建立所謂的**計算圖**。這個圖定義了如何用給定的輸入參數計算輸出（通常是損失函數），並且如果有 GPU 可用，可以將計算推送到 GPU 上執行。框架中有函數可以對計算圖求導並計算梯度，這些梯度可用於優化模型參數。

**高階 API**則將神經網路視為**一連串的層**，使得構建大多數神經網路變得更簡單。訓練模型通常只需準備資料，然後呼叫 `fit` 函數即可完成。

高階 API 讓你能快速構建典型神經網路，而不必擔心太多細節。與此同時，低階 API 提供更多對訓練過程的控制，因此在研究新神經網路架構時被廣泛使用。

同時也要理解，你可以同時使用兩種 API，例如用低階 API 開發自己的網路層架構，然後在用高階 API 建構和訓練的較大網路中使用它。或者你可以用高階 API 定義一個層序列的網路，再用自己的低階訓練迴圈來進行優化。兩種 API 都基於相同的基本概念，且設計上能很好地協同工作。

## 學習

本課程中，我們提供 PyTorch 和 TensorFlow 兩種框架的大部分內容。你可以選擇自己偏好的框架，並只學習對應的筆記本。如果不確定選哪個框架，可以參考網路上關於 **PyTorch vs. TensorFlow** 的討論，也可以兩者都試試看以加深理解。

在可能的情況下，我們會使用高階 API 以簡化學習過程。但我們認為從基礎理解神經網路的運作很重要，因此一開始會從低階 API 和張量開始學習。不過，如果你想快速上手，不想花太多時間在細節上，也可以直接跳到高階 API 的筆記本。

## ✍️ 練習：框架

繼續學習以下筆記本：

低階 API | TensorFlow+Keras 筆記本 | PyTorch
---------|----------------------------|---------
高階 API | Keras                     | *PyTorch Lightning*

掌握框架後，我們來回顧過擬合的概念。

# 過擬合

過擬合是機器學習中非常重要的概念，理解它至關重要！

考慮以下用 5 個點（圖中以 `x` 表示）進行擬合的問題：

!linear | overfit
-------------------------|--------------------------
**線性模型，2 個參數** | **非線性模型，7 個參數**
訓練誤差 = 5.3 | 訓練誤差 = 0
驗證誤差 = 5.1 | 驗證誤差 = 20

* 左圖中，我們看到一條不錯的直線擬合。由於參數數量適中，模型能正確捕捉點的分布趨勢。
* 右圖中，模型過於強大。因為只有 5 個點，但模型有 7 個參數，它可以調整到通過所有點，使訓練誤差為 0。然而，這阻礙了模型理解資料背後的正確模式，因此驗證誤差非常高。

在模型的複雜度（參數數量）和訓練樣本數量之間取得平衡非常重要。

## 過擬合發生的原因

  * 訓練資料不足
  * 模型過於強大
  * 輸入資料中噪聲過多

## 如何偵測過擬合

如上圖所示，過擬合可由非常低的訓練誤差和很高的驗證誤差判斷。通常在訓練過程中，訓練和驗證誤差都會下降，但某個時候驗證誤差可能停止下降並開始上升。這是過擬合的訊號，也表示我們應該停止訓練（或至少保存模型快照）。

過擬合

## 如何防止過擬合

如果發現過擬合，可以採取以下措施：

 * 增加訓練資料量
 * 降低模型複雜度
 * 使用正則化技術，例如稍後會介紹的 Dropout

## 過擬合與偏差-變異權衡

過擬合其實是統計學中更通用問題——偏差-變異權衡（Bias-Variance Tradeoff）的一種情況。考慮模型可能的誤差來源，有兩種誤差：

* **偏差誤差**是因為演算法無法正確捕捉訓練資料間的關係，通常是模型能力不足（**欠擬合**）造成的。
* **變異誤差**是模型擬合了輸入資料中的噪聲，而非有意義的關係（**過擬合**）造成的。

訓練過程中，偏差誤差會下降（模型學會擬合資料），而變異誤差會上升。重要的是要在適當時機停止訓練——無論是手動（偵測到過擬合時）或自動（引入正則化）——以避免過擬合。

## 總結

本課程中，你學到了兩大熱門 AI 框架 TensorFlow 和 PyTorch 的不同 API，以及一個非常重要的主題：過擬合。

## 🚀 挑戰

在附帶的筆記本中，底部會有「任務」；請完成筆記本中的練習任務。

## 複習與自學

請自行研究以下主題：

- TensorFlow
- PyTorch
- 過擬合

並思考以下問題：

- TensorFlow 和 PyTorch 有什麼差異？
- 過擬合和欠擬合有什麼不同？

## 作業

本實驗要求你使用 PyTorch 或 TensorFlow，利用單層和多層全連接網路解決兩個分類問題。

**免責聲明**：  
本文件係使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們致力於確保翻譯的準確性，但請注意，自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應視為權威來源。對於重要資訊，建議採用專業人工翻譯。我們不對因使用本翻譯而產生的任何誤解或誤釋負責。