---
description: log管理工具
---

# Serilog & Datalust-Seq

使用Serilog\(比Nlog效能較佳\)並搭配Seq平台，可有效管理log紀錄

![](../.gitbook/assets/image%20%2831%29.png)

## Serilog

### 準備測試用專案\(可略\)

測試用專案\(ASP.Net Core Web應用程式\) \(Serilog不僅只支援ASP.Net Core\)

![](../.gitbook/assets/image%20%2884%29.png)

![](../.gitbook/assets/image%20%28114%29.png)

![](../.gitbook/assets/image%20%28161%29.png)

建置，確認網站可正常運作

![](../.gitbook/assets/image%20%2890%29.png)

![](../.gitbook/assets/image%20%28219%29.png)

### 安裝Serilog套件

測試專案為asp.net.core專案，因此安裝Serilog.AspNetCore套件

![](../.gitbook/assets/image%20%2855%29.png)

### 設定Log

以Asp.Net Core專案為例，需要在Web Host啟動前建立log物件，並加入相關log

修改Program.cs

![](../.gitbook/assets/image%20%28203%29.png)

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

![](../.gitbook/assets/image%20%2889%29.png)

因此log會被輸出至console介面，為方便測試，可以在專案資料中開啟命令提示字元\(cmd\)，並輸入dotnet run

![](../.gitbook/assets/image%20%28217%29.png)

輸入後，可以看到console中出現相關log即表示

![](../.gitbook/assets/image%20%28256%29.png)

實際進去網站後，再看log，可以發現該套件會寫入非常完整的log紀錄，並有View / Action執行的時間

![](../.gitbook/assets/image%20%28184%29.png)

若要在Controller中自行加入log訊息，直接用logger物件即可，例如以下

![](../.gitbook/assets/image%20%2872%29.png)

```csharp
        public IActionResult Index()
        {
            _logger.LogInformation("TEST-0001 {0}","12345");
            return View();
        }
```

測試進入首頁，可以看到logger被正確寫入，其中若為傳入參數值，serilog會用不同顏色顯示

![](../.gitbook/assets/image%20%289%29.png)

Serilog相關操作說明可參照官網：[https://github.com/serilog/serilog/wiki/Getting-Started](https://github.com/serilog/serilog/wiki/Getting-Started)

### 匯整Log至Datalust - Seq

Datalust - Seq，這個平台可以將log以xml方式儲存並支援sql語法查詢

要將Log寫入Seq系統，需在專案中安裝Serilog.Sinks.Seq套件

![](../.gitbook/assets/image%20%2816%29.png)

修改專案Program.cs - 新增WriteTo.Seq 設定將Log寫入Seq系統中 \(Seq系統預設網址為localhost:5341\)

![](../.gitbook/assets/image%20%28225%29.png)

安裝Datalust Seq平台\(服務\)

Seq官網：[https://datalust.co/seq](https://datalust.co/seq)

點選Download即可\(有Docker Images 或Windows安裝檔\)

![](../.gitbook/assets/image%20%28129%29.png)

若為Windows環境可以直接用windows安裝檔即可，安裝完後會在該電腦上起Seq服務

![](../.gitbook/assets/image%20%28193%29.png)

安裝完畢後，第一次啟動時，需要設定網址及Log存放位置

![](../.gitbook/assets/image%20%28231%29.png)

設定完畢後，可以登入Seq的網頁\(即localhost:5341\) \(目前無log\)

![](../.gitbook/assets/image%20%28116%29.png)

透過dotnet run指令再次將測試專案啟動

![](../.gitbook/assets/image%20%28125%29.png)

Seq介面

![](../.gitbook/assets/image%20%2878%29.png)

可直接下sql或點選右方Queries

![](../.gitbook/assets/image%20%2864%29.png)

Seq有內建Dash Board平台

### ASP.Net WebForm + Serilog

ASP.Net Webform要使用Serilog時，目前沒有像ASP.Net Core有實作好的套件可以在每個事件紀錄log

因此需要自行依需求實作，但難度不高，以下做簡單Demo

#### 1.專案建立

以VS2017/2019建立ASP.Net應用程式\(非.Net Core\)

![](../.gitbook/assets/image%20%2826%29.png)

選擇Web Form\(MVC也可以\)

![](../.gitbook/assets/image%20%28153%29.png)

#### 2.安裝Serilog.Sinks.Seq套件

專案建立完成後，需安裝Serilog.Sink.Seq套件\(可以使用Serilog並直接寫入log至Seq平台\)

![](../.gitbook/assets/image%20%28214%29.png)

Search - Serilog.Sinks.Seq套件，安裝最新版本即可

![](../.gitbook/assets/image%20%28233%29.png)

#### 測試Serilog

為方便測試，先在About頁面，加入一個Button

![](../.gitbook/assets/image%20%28166%29.png)

接著在Button1 - Click事件中寫入Log

![](../.gitbook/assets/image%20%28167%29.png)

基本Log紀錄功能如下

![](../.gitbook/assets/image%20%28175%29.png)

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using Serilog;

namespace SerilogDemo
{
    public partial class About : Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {

        }

        protected void Button1_Click(object sender, EventArgs e)
        {
            //Log建立及設定
            ILogger logger = new LoggerConfiguration()
            .MinimumLevel.Information() //定義最小Log等級
            .WriteTo.Seq("http://localhost:5341") //定義Seq Server位置
            .CreateLogger();

            //寫入Log
            logger.Information("Serilog Demo");

        }
    }
}
```

測試 - 點選Button

![](../.gitbook/assets/image%20%28207%29.png)

進入Seq Server：可以看到Log被寫入具Level為Information，但Event中參數為空

實務上，log中通常會帶入相關參數\(例如http Request Status、User ID、或是其它自定義參數\)

![](../.gitbook/assets/image%20%28236%29.png)

