# IIS Deployment

ASP.Net Core Web應用程式在IIS佈署之注意事項

假設網站實體位置如下：c:\Sample

![](../../.gitbook/assets/image%20%28184%29.png)

在Visual Studio中，點選專案→發佈

![](../../.gitbook/assets/image%20%28192%29.png)

點選「資料夾」，選擇c:\Sample，按下發佈即可

![](../../.gitbook/assets/image%20%2844%29.png)

發佈完成後，至該資料夾如下：程式碼、html部分會被建立在dll檔中

![](../../.gitbook/assets/image%20%2824%29.png)

IIS中，新增應用程式集：.NET CLR版本選擇**沒有受控碼**

![](../../.gitbook/assets/image%20%28146%29.png)

新增網站

![](../../.gitbook/assets/image%20%2879%29.png)

![](../../.gitbook/assets/image%20%2835%29.png)

測試網站：localhost:100

若出現錯誤500.19即表示未安裝.net core host相關套件 

路徑：[https://www.microsoft.com/net/permalink/dotnetcore-current-windows-runtime-bundle-installer](https://www.microsoft.com/net/permalink/dotnetcore-current-windows-runtime-bundle-installer)

測試ok

![](../../.gitbook/assets/image%20%28181%29.png)

## 「錯誤500.19」補充：

若安裝完host套件，仍出現錯誤500.19，可以參考微軟官網建議：在該網站資料-安全性-新增IIS AppPool\應用程式集名稱之使用者，給予讀寫權限

以上例即新增使用者IIS AppPool\Test之權限



