---
description: log管理工具
---

# Serilog & Datalust-Seq

使用Serilog\(比Nlog效能較佳\)並搭配Seq平台，可有效管理log紀錄

![](../.gitbook/assets/image%20%2831%29.png)

## Serilog

### 準備測試用專案\(可略\)

測試用專案\(ASP.Net Core Web應用程式\) \(Serilog不僅只支援ASP.Net Core\)

![](../.gitbook/assets/image%20%2886%29.png)

![](../.gitbook/assets/image%20%28118%29.png)

![](../.gitbook/assets/image%20%28167%29.png)

建置，確認網站可正常運作

![](../.gitbook/assets/image%20%2892%29.png)

![](../.gitbook/assets/image%20%28225%29.png)

### 安裝Serilog套件

測試專案為asp.net.core專案，因此安裝Serilog.AspNetCore套件

![](../.gitbook/assets/image%20%2857%29.png)

### 設定Log

以Asp.Net Core專案為例，需要在Web Host啟動前建立log物件，並加入相關log

修改Program.cs

![](../.gitbook/assets/image%20%28209%29.png)

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

![](../.gitbook/assets/image%20%2891%29.png)

因此log會被輸出至console介面，為方便測試，可以在專案資料中開啟命令提示字元\(cmd\)，並輸入dotnet run

![](../.gitbook/assets/image%20%28223%29.png)

輸入後，可以看到console中出現相關log即表示

![](../.gitbook/assets/image%20%28263%29.png)

實際進去網站後，再看log，可以發現該套件會寫入非常完整的log紀錄，並有View / Action執行的時間

![](../.gitbook/assets/image%20%28190%29.png)

若要在Controller中自行加入log訊息，直接用logger物件即可，例如以下

![](../.gitbook/assets/image%20%2874%29.png)

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

![](../.gitbook/assets/image%20%28231%29.png)

安裝Datalust Seq平台\(服務\)

Seq官網：[https://datalust.co/seq](https://datalust.co/seq)

點選Download即可\(有Docker Images 或Windows安裝檔\)

![](../.gitbook/assets/image%20%28133%29.png)

若為Windows環境可以直接用windows安裝檔即可，安裝完後會在該電腦上起Seq服務

![](../.gitbook/assets/image%20%28199%29.png)

安裝完畢後，第一次啟動時，需要設定網址及Log存放位置

![](../.gitbook/assets/image%20%28237%29.png)

設定完畢後，可以登入Seq的網頁\(即localhost:5341\) \(目前無log\)

![](../.gitbook/assets/image%20%28120%29.png)

透過dotnet run指令再次將測試專案啟動

![](../.gitbook/assets/image%20%28129%29.png)

Seq介面

![](../.gitbook/assets/image%20%2880%29.png)

可直接下sql或點選右方Queries

![](../.gitbook/assets/image%20%2866%29.png)

Seq有內建Dash Board平台

### ASP.Net WebForm + Serilog

ASP.Net Webform要使用Serilog時，目前沒有像ASP.Net Core有實作好的套件可以在每個事件紀錄log

因此需要自行依需求實作，但難度不高，以下做簡單Demo

#### 1.專案建立

以VS2017/2019建立ASP.Net應用程式\(非.Net Core\)

![](../.gitbook/assets/image%20%2826%29.png)

選擇Web Form\(MVC也可以\)

![](../.gitbook/assets/image%20%28158%29.png)

#### 2.安裝Serilog.Sinks.Seq套件

專案建立完成後，需安裝Serilog.Sink.Seq套件\(可以使用Serilog並直接寫入log至Seq平台\)

![](../.gitbook/assets/image%20%28220%29.png)

Search - Serilog.Sinks.Seq套件，安裝最新版本即可

![](../.gitbook/assets/image%20%28239%29.png)

#### 3.測試Serilog

為方便測試，先在About頁面，加入一個Button

![](../.gitbook/assets/image%20%28172%29.png)

接著在Button1 - Click事件中寫入Log

![](../.gitbook/assets/image%20%28173%29.png)

基本Log紀錄功能如下

![](../.gitbook/assets/image%20%28181%29.png)

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

![](../.gitbook/assets/image%20%28213%29.png)

進入Seq Server：可以看到Log被寫入具Level為Information，但Event中參數為空

實務上，log中通常會帶入相關參數\(例如http Request Status、User ID、或是其它自定義參數\)

![](../.gitbook/assets/image%20%28242%29.png)

#### 4. 新增參數\(Enrich\)

Serilog在建立log物件時，可以用Enrich方法新增參數至log中

例如以下新增了兩個參數，之後用此logger物件寫入的log都會帶此兩參數

```csharp
        protected void Button1_Click(object sender, EventArgs e)
        {
            //Log建立及設定
            ILogger logger = new LoggerConfiguration()
            .MinimumLevel.Information() //定義最小Log等級
            .Enrich.WithProperty("參數1", DateTime.Now.ToShortDateString()) //新增參數1至log
            .Enrich.WithProperty("參數2", sender.ToString()) //新增參數1至log
            .WriteTo.Seq("http://localhost:5341") //定義Seq Server位置
            .CreateLogger();

            //寫入Log
            logger.Information("Serilog Demo");

        }
```

![](../.gitbook/assets/image%20%28245%29.png)

用此方式加入參數，比較適合用在程式名稱這類型參數\(不需動態取得值\)

實務上，logger物件通常只會建立一次\(以下僅示意，logger可以存在session中或memory catch\)

若需要在特定程式區塊動態加入專屬的參數時，較不適用

![](../.gitbook/assets/image%20%2854%29.png)

#### 5.動態加入參數

若有做到動態建立參數，比較直覺的做法為以下

自定義參數集合、訊息內容格式、建立log事件，最後呼叫Write寫入log

```csharp
        protected void Button1_Click(object sender, EventArgs e)
        {
            //To Do：希望加入動態參數3 - 值為Button1

            //實作log方法
            //1.建立參數集合
            List<LogEventProperty> collectedProperties = new List<LogEventProperty>();

            //2.給定參數 (可多筆)
            var properties = collectedProperties.Concat(new[]
           {
                new LogEventProperty("參數3", new ScalarValue("Button1")),
                new LogEventProperty("參數4", new ScalarValue("12345"))
            });

            //3.宣告log訊息格式
            MessageTemplate _messageTemplate = new MessageTemplateParser().Parse("SerilogDemo - {參數3} Click");
            //4.write log
            var evt = new LogEvent(DateTimeOffset.Now, LogEventLevel.Information, null, _messageTemplate, properties);
            logger.Write(evt);
        }
```

測試結果：

![](../.gitbook/assets/image%20%28151%29.png)

#### 6.情境實作 - 基本網站/系統需要的log功能

先定義需要紀錄的項目\(參數\)：

* [x] 網址\(實際網址\)
* [x] 程式名稱\(方便辨別\)
* [x] 使用者名稱
* [x] 事件或方法名稱
* [x] Trace ID\(同一次回應的相關log需要給定相同的Trace ID\)
* [x] Cost\(花費時間\) \(Optional\)

1. 建立LogHelper類別 - 將實作的方法都寫在這個類別中

![](../.gitbook/assets/image%20%2856%29.png)

寫入基本功能，包含固定的參數 - 網址、程式名稱、使用者名稱 \(可以在page\_load事件直接定義\)

並先給定一個information方法，呼叫原生information方法

![](../.gitbook/assets/image%20%28103%29.png)

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;

using System.Web.UI;
using Serilog;
using Serilog.Events;
using Serilog.Parsing;

namespace SerilogDemo
{
    public class LogHelper
    {
        private ILogger _logger;

        /// <summary>
        /// 
        /// </summary>
        /// <param name="programName">程式名稱</param>
        /// <param name="page">page物件(為了取得網址)</param>
        public LogHelper(string programName, Page page)
        {
            _logger = new LoggerConfiguration()
            .MinimumLevel.Debug()
            .Enrich.WithProperty("網址", page.Request.Url.OriginalString)
            .Enrich.WithProperty("程式名稱", programName)
            .Enrich.WithProperty("使用者名稱", HttpContext.Current.User.Identity.Name)
            .Enrich.FromLogContext()
            .WriteTo.Seq("http://localhost:5341")
            .CreateLogger();
        }

        public void Information(string sMessage)
        {
            _logger.Information(sMessage);
        }
    }
}
```

2. 使用LogHelper寫入log

在About頁面，Page Load事件，將LogHelper物件建立並傳入程式名稱及Page物件 \(用Session存起來\)

Button1\_Click事件中，直接使用LogHelper中的Information方法寫入log

![](../.gitbook/assets/image%20%2899%29.png)

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using Serilog;
using Serilog.Events;
using Serilog.Parsing;

namespace SerilogDemo
{
    public partial class About : Page
    {

        private LogHelper log;

        protected void Page_Load(object sender, EventArgs e)
        {
            if (!IsPostBack)
            {
                log = new LogHelper("About", this); //宣告LogHelper物件(程式名稱，page物件)
                Session["log"] = log; //紀錄至Session
            }
            else
            {
                log = (LogHelper)Session["log"];
            }
        }

        protected void Button1_Click(object sender, EventArgs e)
        {
            try
            {
                log.Information("Serilog Demo");
            }
            catch (Exception ex)
            {
                //To Do Exception log
            }
        }
    }
}
```

測試結果：

其中使用者名稱為空是正常現象，因為這個Demo網站沒有做登入功能，所以HttpContext.Current.User.Identity.Name會為空值

![](../.gitbook/assets/image%20%28163%29.png)



