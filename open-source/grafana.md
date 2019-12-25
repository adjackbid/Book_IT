---
description: 開源Dash Board平台 (Windows / Linux / Container)
---

# Grafana

Grafana為開源的Dash Board管理平台\(Go語言編寫\)，可動態建立資料來源、看板、各類Chart圖及權限控管\(AD\(LDAP\)、表單認證\)

官網：[https://grafana.com/grafana/](https://grafana.com/grafana/)

![](../.gitbook/assets/image%20%28101%29.png)

## Grafana架設方式

Grafana有提供安裝程式，可於官網中Download頁面下載安裝檔

![](../.gitbook/assets/image%20%28235%29.png)

Windows安裝時，可直接下載打包好的安裝檔即可

![](../.gitbook/assets/image%20%28154%29.png)

下載完畢後，直接點選安裝檔

![](../.gitbook/assets/image%20%28193%29.png)

安裝完畢後，至安裝目錄中\GrafanaLabs\grafana\conf資料下，copy一份sample.ini，並改名為custom.ini\(系統設置檔\)

![](../.gitbook/assets/image%20%285%29.png)

該檔案中，可設定進行相關設定\(例如認證方式、LDAP設定等\)

主要需要調整的項目為server區的http\_port，預設值為3000，可依需求調整該值\(例如80等\)

![](../.gitbook/assets/image%20%2881%29.png)

設定完畢後，執行bin資料夾下grafana-server.exe即可啟動網址服務

![](../.gitbook/assets/image%20%2835%29.png)

![](../.gitbook/assets/image%20%28122%29.png)

服務啟動後，即可登入網站

## 建立Data Source

登入後，首先需建立Data Source - 內建支援MySql、PostgreSQL、SQL Server

註：Oracle需使用Grafana Enterprise版本才可使用

![](../.gitbook/assets/image%20%28134%29.png)

![](../.gitbook/assets/image%20%28105%29.png)

設定host、db名稱、使用者帳號等資訊後，即可

![](../.gitbook/assets/image%20%2813%29.png)

## 建立Dash Board

接著即可建立Dash Board

![](../.gitbook/assets/image%20%28246%29.png)

點選Choose Visualization - 選擇圖表類型

![](../.gitbook/assets/image%20%2848%29.png)

選擇Data Source - 調整SQL

註：圖表多數會使用到TIME、VALUE、METRIC欄位\(Series名稱\)

![](../.gitbook/assets/image%20%28151%29.png)

例如以下：

![](../.gitbook/assets/image%20%2851%29.png)

若不想要X軸為時間的話，可以在圖表設定中調整X軸模式為Series，則可以畫出長條圖，以Metric為一個X Value

![](../.gitbook/assets/image%20%28133%29.png)

若有兩個以上資料來源亦可在Data Source頁面中，新增Query，並輸入SQL，即可畫出以下圖示

![](../.gitbook/assets/image%20%28157%29.png)

圖表亦可以新增管制上下限

![](../.gitbook/assets/image%20%28239%29.png)

![](../.gitbook/assets/image%20%2878%29.png)

最後按下Save即可\(右上方\)

![](../.gitbook/assets/image%20%2819%29.png)

一個Dash Board可以新增多個圖表\(Panel\)，大小、位置可自行拖拉

![](../.gitbook/assets/image%20%28211%29.png)



## Plugins安裝

若預設的圖表或功能不足時，可以至官網plugin中下載需要的圖表或功能\(例如Oracle Data Source支援\)

![](../.gitbook/assets/image%20%28107%29.png)

圖表區會在Panel類別

![](../.gitbook/assets/image%20%28229%29.png)

點選欲安裝的Plugin，例如Pie Chart。頁面中會有簡介及安裝說明\(一些元件會寫在Installation\)

通常都會有grafana-cli指令，例如以下：grafana-cli plugins install grafana-piechart-panel

![](../.gitbook/assets/image%20%28139%29.png)

grafana-cli程式放置在bin資料夾下

![](../.gitbook/assets/image%20%2839%29.png)

進到bin資料夾後，將上方的位置列修改成cmd，即可快速進到cmd並位置於該資料夾下

![](../.gitbook/assets/image%20%28155%29.png)

貼上cli指令→按下enter即可

![](../.gitbook/assets/image%20%2846%29.png)

安裝完畢後，服務需要重啟才會生效，可至工作管理員→服務→grafana點選重新啟動即可

![](../.gitbook/assets/image%20%28240%29.png)

