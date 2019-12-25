# Serilog & Datalust-Seq

## Serilog

### 準備測試用專案\(可略\)

測試用專案\(ASP.Net Core Web應用程式\)

![](../.gitbook/assets/image%20%2882%29.png)

![](../.gitbook/assets/image%20%28112%29.png)

![](../.gitbook/assets/image%20%28158%29.png)

建置，確認網站可正常運作

![](../.gitbook/assets/image%20%2888%29.png)

![](../.gitbook/assets/image%20%28211%29.png)

### 安裝Serilog套件

測試專案為asp.net.core專案，因此安裝Serilog.AspNetCore套件

![](../.gitbook/assets/image%20%2853%29.png)

### 設定Log

以Asp.Net Core專案為例，需要在Web Host啟動前建立log物件，並加入相關log

修改Program.cs

![](../.gitbook/assets/image%20%28197%29.png)

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using Serilog;
using Serilog.Events;

namespace Web1
{
    public class Program
    {
        public static void Main(string[] args)
        {
            //CreateHostBuilder(args).Build().Run();

            Log.Logger = new LoggerConfiguration()
            .MinimumLevel.Debug()
            .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
            .Enrich.FromLogContext()
            .WriteTo.Console()
            .CreateLogger();

            try
            {
                Log.Information("Starting web host");
                CreateHostBuilder(args).Build().Run();
            }
            catch (Exception ex)
            {
                Log.Fatal(ex, "Host terminated unexpectedly");
            }
            finally
            {
                Log.CloseAndFlush();
            }
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    //webBuilder.UseStartup<Startup>();
                    webBuilder.UseStartup<Startup>().UseSerilog();
                });
    }
}

```

因目前是將log輸出至console - 如下圖\(WriteTo.Console\)

![](../.gitbook/assets/image%20%2887%29.png)

因此log會被輸出至console介面，為方便測試，可以在專案資料中開啟命令提示字元\(cmd\)，並輸入dotnet run

![](../.gitbook/assets/image%20%28209%29.png)

輸入後，可以看到console中出現相關log即表示

![](../.gitbook/assets/image%20%28246%29.png)

實際進去網站後，再看log，可以該套件會寫入非常完整的log紀錄，並有View / Action執行的時間

![](../.gitbook/assets/image%20%28178%29.png)

若要在Controller中自行加入log訊息，直接用logger物件即可，例如以下

![](../.gitbook/assets/image%20%2870%29.png)

```csharp
        public IActionResult Index()
        {
            _logger.LogInformation("TEST-0001 {0}","12345");
            return View();
        }
```

測試進入首頁，可以看到logger被正確寫入，其中若為傳入參數值，serilog會用不同顏色顯示

![](../.gitbook/assets/image%20%289%29.png)

### 匯整Log至Datalust - Seq

Datalust - Seq，這個平台可以將log以xml方式儲存並支援sql語法查詢

要將Log寫入Seq系統，需在專案中安裝Serilog.Sinks.Seq套件

![](../.gitbook/assets/image%20%2816%29.png)

修改專案Program.cs - 新增WriteTo.Seq 設定將Log寫入Seq系統中 \(Seq系統預設網址為localhost:5341\)

![](../.gitbook/assets/image%20%28217%29.png)

安裝Datalust Seq平台\(服務\)

Seq官網：[https://datalust.co/seq](https://datalust.co/seq)

點選Download即可\(有Docker Images 或Windows安裝檔\)

![](../.gitbook/assets/image%20%28127%29.png)

若為Windows環境可以直接用windows安裝檔即可，安裝完後會在該電腦上起Seq服務

![](../.gitbook/assets/image%20%28187%29.png)

安裝完畢後，第一次啟動時，需要設定網址及Log存放位置

![](../.gitbook/assets/image%20%28223%29.png)

設定完畢後，可以登入Seq的網頁\(即localhost:5341\) \(目前無log\)

![](../.gitbook/assets/image%20%28114%29.png)

透過dotnet run指令再次將測試專案啟動

![](../.gitbook/assets/image%20%28123%29.png)

Seq介面

![](../.gitbook/assets/image%20%2876%29.png)

可直接下sql或點選右方Queries

![](../.gitbook/assets/image%20%2862%29.png)

Seq有內建Dash Board平台

![](../.gitbook/assets/image%20%28166%29.png)

