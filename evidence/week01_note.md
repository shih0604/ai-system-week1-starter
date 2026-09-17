# Week 1 Evidence

學號：7115029024
姓名：蘇詩晴

## 1. Dataset 在哪裡？

`data/customer_intent_demo.csv`

本次實驗使用客服問題意圖分類資料集，共有 100 筆資料，
包含 4 種意圖類別，每個類別各 25 筆。

---

## 2. Baseline 在哪裡？

`src/rule_baseline.py`

本次使用 Rule-based Baseline，
透過預先設定的文字規則判斷客服訊息所屬的問題類別。

---

## 3. 本次 Accuracy

Accuracy = 0.950

代表 100 筆資料中，有 95 筆被 Baseline 正確分類，
另外有 5 筆分類錯誤。

---

## 4. Failure Case

Input：

`訂單取消後退款多久會入帳`

Ground Truth：

`refund_return`

Baseline Prediction：

`order_delivery`

---

## 5. 為什麼 Baseline 會錯？

因為rule_baseline函式先偵測到了"訂單"這個關鍵字，而將它分類到order_delivery，但這個問題的重點關鍵字其實是後面的"取消"、"退款"，主要的語意是偏向refund_return這個類別。

---

## 6. 這是否代表現在一定要使用更複雜的 AI？為什麼？

不一定。

現在的準確率已經算很高了，可以先從修改rule_baseline的關鍵詞規範，如果之後遇到更多更複雜冗長、較難判斷真正語意的問題，沒辦法依現行的規則去分類大部分的問題後，再考慮使用更複雜的 AI。