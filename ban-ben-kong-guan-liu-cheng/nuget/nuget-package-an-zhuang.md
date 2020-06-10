---
description: 示範Visual Studio專案，從自架的Nuget Server安裝Nuget套件
---

# Nuget Package安裝

首先Copy Nuget Server網站上的Package Repository URL：http://localhost/NugetServer/Nuget

![](../../.gitbook/assets/image%20%281%29.png)

Visual Studio中工具→Nuget套件管理員→套件管理員設定 \(VS2013以後應該都有此選項\)

![](../../.gitbook/assets/image%20%2847%29.png)

在NuGet套件管理員→套件來源中新增http://localhost/NugetServer/Nuget

![](../../.gitbook/assets/image%20%28114%29%20%281%29.png)

設定完畢後，即可點選Nuget套件管理員→管理方案的Nuget套件

![](../../.gitbook/assets/image%20%28204%29.png)

開啟後，在套件來源選擇處，將來源改成剛才所的來源\(例如CIM\)

![](../../.gitbook/assets/image%20%28183%29.png)

選取完畢後，即可看到該Server中目前可使用的Nuget套件

![](../../.gitbook/assets/image%20%28143%29%20%281%29.png)

點選套件按下安裝即可

![](../../.gitbook/assets/image%20%2830%29%20%282%29.png)

安裝完畢，套件將會被include至專案並產生packages.config檔

![](../../.gitbook/assets/image%20%2835%29%20%281%29.png)

其中packages.config檔，主要紀錄該專案已安裝的套件清單\(包含版本\)

![](../../.gitbook/assets/image%20%282%29%20%282%29.png)

未來程式碼進行版本時，套件中的dll檔則不需加入版本，僅需將packages.config加入版控即可

Visual Studio在專案建置時，會自動從Server上抓取相關套件\(若目前專案沒有該套件相關元件檔時\)

註：專案引用/安裝的套件若有版本更新時，若要手動重新安裝新版

例如以下：

![](../../.gitbook/assets/image%20%2872%29%20%281%29.png)

