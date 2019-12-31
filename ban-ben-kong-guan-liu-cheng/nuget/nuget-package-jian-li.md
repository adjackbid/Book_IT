---
description: 如何將自製的dll及第三方套件打包成一個Package讓其它專案可以直接使用
---

# Nuget Package建立

簡易做法，使用Nuget Package Explorer套件

{% embed url="https://github.com/NuGetPackageExplorer/NuGetPackageExplorer" %}

Windows可以接直在Microsoft商店中直接下載安裝

![https://www.microsoft.com/zh-tw/p/nuget-package-explorer/9wzdncrdmdm3?activetab=pivot:overviewtab](../../.gitbook/assets/image%20%2812%29.png)

安裝完畢後，在開始→工作列中可以開啟程式

![](../../.gitbook/assets/image%20%2829%29.png)

開啟後，點選「Create a new package」

![](../../.gitbook/assets/image%20%28232%29.png)

在Package metadata區塊中，可以編輯套件的資訊\(名稱、版本、作者、相依性\)

點選編輯圖示即可以修改

![](../../.gitbook/assets/image%20%2872%29.png)

將id、版本、title等資訊修改好，按下確認即可

![](../../.gitbook/assets/image%20%2893%29.png)

接著要將自製的dll檔打包至這個Package中

點選Content→Add→Lib Folder

![](../../.gitbook/assets/image%20%2810%29.png)

新增完lib資料夾後，如下圖，該資料可以放至需要被安裝至專案的dll檔

![](../../.gitbook/assets/image%20%2851%29.png)

在lib資料上按下右鍵→Add.NET folder→選擇版本\(若有多版本情下，可以建立多筆或是選擇no version不分版本\)

![](../../.gitbook/assets/image%20%2894%29.png)

接著在新建的folder中加入dll檔

![](../../.gitbook/assets/image%20%2839%29.png)

![&#x8A3B;&#xFF1A;.pdb&#x6A94;&#x53EF;&#x8996;&#x60C5;&#x6CC1;&#x52A0;&#x5165;&#xFF0C;&#x5176;&#x53EF;&#x8B93;&#x4F7F;&#x7528;&#x7AEF;debug tracing&#x7528;](../../.gitbook/assets/image%20%28123%29.png)

![](../../.gitbook/assets/image%20%28179%29.png)

加入完成後，在存檔前，可以先用工具分析Package是否有設定問題

Tools→Analyze Packge

![](../../.gitbook/assets/image%20%2898%29.png)

Package Analysis出現無任何問題，即可存檔

![](../../.gitbook/assets/image%20%28185%29.png)

File→Save As

![](../../.gitbook/assets/image%20%286%29.png)

![](../../.gitbook/assets/image%20%28117%29.png)

後續將產生的.nupkg檔發佈至Nuget Server上即可使用

![](../../.gitbook/assets/image%20%2878%29.png)

