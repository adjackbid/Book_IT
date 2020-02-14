---
description: Toad好用但少人知道的功能...
---

# Oracle Toad Tips

## Auto Replace

View → Toad Options → Editor - Auto Replacement

![](.gitbook/assets/image%20%2815%29.png)

![](.gitbook/assets/image%20%28138%29.png)

可自行新增Auto Replace組合：例如輸入sfr自動取代成select \* from，可以設定如下

![](.gitbook/assets/image%20%28239%29.png)

輸入完sfr按下空格鍵後

![](.gitbook/assets/image%20%28385%29.png)

將依設定自動取代成select \* from

![](.gitbook/assets/image%20%28393%29.png)

常用的情境為有些欄位很長，而且很常打，可以把它設成Auto Replace，例如交易時間\(TransactionTime\)

可以設定成tst

![](.gitbook/assets/image%20%28466%29.png)

![](.gitbook/assets/image%20%28228%29.png)

## Code Templates

Toad有類似Code Snippets功能，可以將常用的程式碼片段包裝成Code Templates

例如以下Sql Join Test、Test2，取得Test2表格中相關資訊，可以建立成Code Templates並將t1.A設成變數

![](.gitbook/assets/image%20%28250%29.png)

View → Toad Options → Behavior - Code templates

![](.gitbook/assets/image%20%28391%29.png)

![](.gitbook/assets/image%20%28330%29.png)

點選Add

![](.gitbook/assets/image%20%28164%29.png)

建立新的Code Template：輸入簡碼及說明

![](.gitbook/assets/image%20%28150%29.png)

新增完後，點選該簡碼並在下方空白處，新增寫入對應的Code，其中變數可以用&加變數名稱

![](.gitbook/assets/image%20%28361%29.png)

建立完成後，需調整Code Template呼叫的熱鍵 \(因為預設為Ctrl + Space 會與輸入法切換相衝\)

View → Toad Options → Editor - Behavior - Key mapping

![](.gitbook/assets/image%20%28400%29.png)

找到Code templates popup，將Primary由Ctrl + Space改成想要的熱鍵\(例如Ctrl + Q\)

![](.gitbook/assets/image%20%28232%29.png)

之後只要輸入完Code Template的簡碼後，再按下Ctrl + Q，即會跳出對應的Code Template

![](.gitbook/assets/image%20%28112%29.png)

如果有變數時，會跳出變數輸入的畫面，輸入完後，按下Enter\(或OK\)即可

![](.gitbook/assets/image%20%28176%29.png)

![](.gitbook/assets/image%20%28373%29.png)

部門內應用：可以將部門內常用的Code Template建立好，在匯出成檔案，讓其它人匯入，即可讓新進人員快速上手

![](.gitbook/assets/image%20%2855%29.png)

## SQL Recall

View → SQL Recall → History

![](.gitbook/assets/image%20%28210%29.png)

曾經下過的SQL、執行時間

![](.gitbook/assets/image%20%28205%29.png)



