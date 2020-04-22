---
description: 如何在內部架設Nuget Server
---

# Nuget Server建置

Nuget Server建置可以透過Visual Studio建立網頁應用程式，再部署在IIS上即可

以下說明內容包含：

* Nuget Server專案建置
* IIS部署注意事項

VS2017/VS2019建立ASP網頁應用程式\(空白\)

以下以VS2019為例

![](../../.gitbook/assets/image%20%2876%29.png)

![](../../.gitbook/assets/image%20%28490%29.png)

![](../../.gitbook/assets/image%20%28326%29.png)

安裝Nuget.Server套件 \(工具=&gt;Nuget套件管理員=&gt;管理方案的Nuget套件\)

![](../../.gitbook/assets/image%20%28371%29.png)

輸入Nuget.Server

![](../../.gitbook/assets/image%20%28304%29.png)

點選安裝

![](../../.gitbook/assets/image%20%28454%29.png)

安裝完畢，Nuget Server物件將自動產生如下：

![](../../.gitbook/assets/image%20%28211%29.png)

發佈網站至IIS資料夾

![](../../.gitbook/assets/image%20%2880%29.png)

![](../../.gitbook/assets/image%20%28464%29.png)

註：需將NugetServer根目錄下Packages資料權限進行調整

![](../../.gitbook/assets/image%20%28130%29.png)

需給予IIS\_IUSR寫入權限\(以避免未來使用時，權限不足問題\)

![](../../.gitbook/assets/image%20%2848%29.png)

Server建置完後，即可測試，例如：http://localhost/NugetServer

![](../../.gitbook/assets/image%20%281%29.png)

網站服務啟動成功即可看到上圖畫面，其中Package Source網址即Nuget Repository Server網址

