---
description: log管理工具
---

# Serilog & Datalust-Seq

Serilog：功能類似Nlog，但效能較佳

{% embed url="https://serilog.net/" %}

![](../.gitbook/assets/image%20%28246%29.png)

Datalust Seq平台：可自架，以檔案方式存儲log，但可以用類似SQL語法查詢log紀錄

並提供Dashboard、alarm等功能\(可搭配Serilog使用\)

{% embed url="https://datalust.co/seq" %}

![](../.gitbook/assets/image%20%2830%29%20%281%29.png)

## Serilog

### 準備測試用專案\(可略\)

測試用專案\(ASP.Net Core Web應用程式\) \(Serilog不僅只支援ASP.Net Core\)

![](../.gitbook/assets/image%20%2882%29.png)

![](../.gitbook/assets/image%20%28112%29.png)

![](../.gitbook/assets/image%20%28158%29%20%281%29.png)

建置，確認網站可正常運作

![](../.gitbook/assets/image%20%2888%29%20%281%29.png)

![](../.gitbook/assets/image%20%28211%29.png)

### 安裝Serilog套件

測試專案為asp.net.core專案，因此安裝Serilog.AspNetCore套件

![](../.gitbook/assets/image%20%2853%29%20%281%29.png)

### 設定Log

以Asp.Net Core專案為例，需要在Web Host啟動前建立log物件，並加入相關log

修改Program.cs

![](../.gitbook/assets/image%20%28197%29%20%281%29.png)

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

![](../.gitbook/assets/image%20%2887%29%20%281%29.png)

因此log會被輸出至console介面，為方便測試，可以在專案資料中開啟命令提示字元\(cmd\)，並輸入dotnet run

![](../.gitbook/assets/image%20%28209%29.png)

輸入後，可以看到console中出現相關log即表示

![](../.gitbook/assets/image%20%28246%29.png)

實際進去網站後，再看log，可以發現該套件會寫入非常完整的log紀錄，並有View / Action執行的時間

![](../.gitbook/assets/image%20%28178%29%20%281%29.png)

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

Serilog相關操作說明可參照官網：[https://github.com/serilog/serilog/wiki/Getting-Started](https://github.com/serilog/serilog/wiki/Getting-Started)

### 安裝Datalust - Seq

Datalust - Seq，這個平台可以將log以xml方式儲存並支援sql語法查詢

要將Log寫入Seq系統，需在專案中安裝Serilog.Sinks.Seq套件

![](../.gitbook/assets/image%20%2816%29.png)

修改專案Program.cs - 新增WriteTo.Seq 設定將Log寫入Seq系統中 \(Seq系統預設網址為localhost:5341\)

![](../.gitbook/assets/image%20%28217%29%20%281%29.png)

安裝Datalust Seq平台\(服務\)

Seq官網：[https://datalust.co/seq](https://datalust.co/seq)

點選Download即可\(有Docker Images 或Windows安裝檔\)

![](../.gitbook/assets/image%20%28127%29%20%281%29.png)

若為Windows環境可以直接用windows安裝檔即可，安裝完後會在該電腦上起Seq服務

![](../.gitbook/assets/image%20%28187%29%20%281%29.png)

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

### ASP.Net WebForm + Serilog

ASP.Net Webform要使用Serilog時，目前沒有像ASP.Net Core有實作好的套件可以在每個事件紀錄log

因此需要自行依需求實作，但難度不高，以下做簡單Demo

註：需使用Datalust - Seq平台，可參考 - [安裝Datalust - Seq](serilog-and-datalust-seq.md#an-zhuang-datalust-seq)

#### 1.專案建立

以VS2017/2019建立ASP.Net應用程式\(非.Net Core\)

![](../.gitbook/assets/image%20%2827%29.png)

選擇Web Form\(MVC也可以\)

![](../.gitbook/assets/image%20%28165%29%20%281%29.png)

#### 2.安裝Serilog.Sinks.Seq套件

專案建立完成後，需安裝Serilog.Sink.Seq套件\(可以使用Serilog並直接寫入log至Seq平台\)

![](../.gitbook/assets/image%20%28233%29.png)

Search - Serilog.Sinks.Seq套件，安裝最新版本即可

![](../.gitbook/assets/image%20%28254%29.png)

#### 3.測試Serilog

為方便測試，先在About頁面，加入一個Button

![](../.gitbook/assets/image%20%28179%29.png)

接著在Button1 - Click事件中寫入Log

![](../.gitbook/assets/image%20%28180%29%20%281%29.png)

基本Log紀錄功能如下

![](../.gitbook/assets/image%20%28189%29%20%281%29.png)

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

![](../.gitbook/assets/image%20%28224%29.png)

進入Seq Server：可以看到Log被寫入具Level為Information，但Event中參數為空

實務上，log中通常會帶入相關參數\(例如http Request Status、User ID、或是其它自定義參數\)

![](../.gitbook/assets/image%20%28257%29.png)

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

![](../.gitbook/assets/image%20%28260%29%20%281%29.png)

用此方式加入參數，比較適合用在程式名稱這類型參數\(不需動態取得值\)

實務上，logger物件通常只會建立一次\(以下僅示意，logger可以存在session中或memory catch\)

若需要在特定程式區塊動態加入專屬的參數時，較不適用

![](../.gitbook/assets/image%20%2856%29.png)

#### 5.動態加入參數

若有做到動態建立參數，有兩個方法

方法1：自定義參數集合、訊息內容格式、建立log事件，最後呼叫Write寫入log

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

![](../.gitbook/assets/image%20%28158%29.png)

方法2\(2020/01/01 Update\)：使用Enrich.FromLogContext\(\)，在寫入log前，先Push參數至LogContext

![](../.gitbook/assets/image%20%28166%29%20%281%29.png)

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
using Serilog.Context;

namespace SerilogDemo
{
    public partial class About : Page
    {
        private ILogger _logger;

        protected void Page_Load(object sender, EventArgs e)
        {
            _logger = new LoggerConfiguration()
            .MinimumLevel.Information()
            .Enrich.WithProperty("參數1","1")
            .Enrich.WithProperty("參數2","2")
            .Enrich.FromLogContext() //使用LogContext加入至事件的參數中
            .WriteTo.Seq("http://localhost:5341")
            .CreateLogger();
        }

        protected void Button1_Click(object sender, EventArgs e)
        {
            LogContext.PushProperty("參數3", "3"); //Push參數1至log參數
            LogContext.PushProperty("參數4", "4"); //Push參數2至log參數
            _logger.Information("Serilog Demo");
        }
    }
}
```

測試結果：

![](../.gitbook/assets/image%20%28273%29.png)

#### 6.情境實作 - 基本網站/系統需要的log功能

先定義需要紀錄的項目\(參數\)：

* [x] 網址\(實際網址\)
* [x] 程式名稱\(方便辨別\)
* [x] 使用者名稱
* [x] 事件或方法名稱
* [x] Trace ID\(同一次回應的相關log需要給定相同的Trace ID\)
* [x] Cost\(花費時間\) \(Optional\)

1. 建立LogHelper類別 - 將實作的方法都寫在這個類別中

![](../.gitbook/assets/image%20%2859%29.png)

寫入基本功能，包含固定的參數 - 網址、程式名稱、使用者名稱 \(可以在page\_load事件直接定義\)

並先給定一個information方法，呼叫原生information方法

![](../.gitbook/assets/image%20%28109%29.png)

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

![](../.gitbook/assets/image%20%28105%29.png)

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

![](../.gitbook/assets/image%20%28170%29.png)

3. 新增取得事件方法參數

為了方便使用動態加入參數功能，新增一個方法WriteLog\(需傳入LogLevel\)\)，裡面實作一個參數-方法名稱

Information方法改寫使用WriteLog

![](../.gitbook/assets/image%20%2857%29%20%281%29.png)

其中取得方法名稱，可以直接使用C\#中的Reflection特性，取得呼叫的來源，以目前情境要往回推2層才能得到呼叫端，因此GetFrame給定2

```csharp
 private void WriteLog(LogEventLevel logEventLevel, string sMessage, Exception ex = null)
        {
            //定義參數集合
            List<LogEventProperty> collectedProperties = new List<LogEventProperty>();

            //取得事件或方法(C# Reflection)
            StackTrace stackTrace = new StackTrace();
            string sRequestMethod = stackTrace.GetFrame(2).GetMethod().Name; //往上兩層

            //給定參數
            var properties = collectedProperties.Concat(new[]
           {
                new LogEventProperty("方法名稱", new ScalarValue(sRequestMethod))
            });

            //訊息格式
            MessageTemplate _messageTemplate = new MessageTemplateParser().Parse(sMessage); //不帶參數，僅顯示訊息

            //write log
            var evt = new LogEvent(DateTimeOffset.Now, logEventLevel, ex, _messageTemplate, properties);
            _logger.Write(evt);
        }

        public void Information(string sMessage)
        {
            WriteLog(LogEventLevel.Information, sMessage);
            //_logger.Information(sMessage);
        }
```

測試結果：

![](../.gitbook/assets/image%20%28190%29%20%281%29.png)

4.針對不同Level log進行處理

Serilog預設有不同層級的EventLevel，因此需針對不同Log層級進行Log寫入實作 

例如以下在LogHelper中加入Error、Warning處理

```csharp
        public void Error(string sMessage, Exception ex = null) //error通常有Exception
        {
            WriteLog(LogEventLevel.Error, sMessage, ex);
        }

        public void Warning(string sMessage)
        {
            WriteLog(LogEventLevel.Warning, sMessage);
        }
```

至About頁面新增exception處理

![](../.gitbook/assets/image%20%28228%29.png)

```csharp
        protected void Button1_Click(object sender, EventArgs e)
        {
            try
            {
                log.Information("Serilog Demo");

                throw new Exception("Test Error"); //手動拋出例外
            }
            catch (Exception ex)
            {
                log.Error("Something wrong", ex); //記錄例外log
            }
        }
```

測試：可以得到兩筆log, 其中Error層級，因為有傳入Exception物件，因此可以得到完整錯誤內容及Trace資訊\(程式碼第幾行\)

![](../.gitbook/assets/image%20%28263%29.png)

5. 新增Trace ID

一筆交易或事件通常會有多筆log，因此需要新增Trace ID以紀錄這些交易屬於同一次交易

以下為簡單實作

在LogHelper中新增TraceID的參數及初始化方法

![](../.gitbook/assets/image%20%28186%29.png)

```csharp
        private string _traceID = "";

        /// <summary>
        /// 產生TraceID
        /// </summary>
        private void InitTraceID()
        {
            _traceID = Guid.NewGuid().ToString();
        }

        /// <summary>
        /// 初始化Log
        /// </summary>
        public void Init()
        {
            InitTraceID();
        }
```

在WriteLog方法中，加入TraceID參數

![](../.gitbook/assets/image%20%2869%29%20%282%29.png)

```csharp
  var properties = collectedProperties.Concat(new[]
  {
       new LogEventProperty("方法名稱", new ScalarValue(sRequestMethod)),
       new LogEventProperty("TraceID", new ScalarValue(_traceID)),
   });
```

About頁面中，Button1 Click事件最上方先初始化Log

![](../.gitbook/assets/image%20%28129%29.png)

```csharp
        protected void Button1_Click(object sender, EventArgs e)
        {
            log.Init(); //初始化：取定TraceID
            try
            {
                log.Information("Serilog Demo");

                throw new Exception("Test Error"); //手動拋出例外
            }
            catch (Exception ex)
            {
                log.Error("Something wrong", ex); //記錄例外log
            }
        }
```

測試：可以得到兩筆log且TraceID為相同值，後續在查詢問題時，可以依此TraceID進行查詢

![](../.gitbook/assets/image%20%2878%29.png)

6. 新增執行時間

有些情況下，需要知道部分交易執行時間\(例如報表或重要交易\)

在LogHelper中，新增兩個變數start / end紀錄開始、結束時間

在Init方法中，紀錄開始時間

![](../.gitbook/assets/image%20%28241%29.png)

為了能寫入一筆log並且有花費時間，在WriteLog方法中加入cost參數

若Cost &gt; 0 時，新增參數花費時間 \(預設為-1\)

![](../.gitbook/assets/image%20%28230%29.png)

新增CompleteLog方法，以紀錄結束時間並寫入Log紀錄

![](../.gitbook/assets/image%20%28278%29.png)

完整LogHelper

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
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

        private string _traceID = "";

        //Log Start / End
        private long start;
        private long end;

        /// <summary>
        /// 產生TraceID
        /// </summary>
        private void InitTraceID()
        {
            _traceID = Guid.NewGuid().ToString();
        }

        /// <summary>
        /// 初始化Log
        /// </summary>
        public void Init()
        {
            start = Stopwatch.GetTimestamp(); //開始時間
            InitTraceID();
        }

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

        private void WriteLog(LogEventLevel logEventLevel, string sMessage, Exception ex = null, decimal cost = -1)
        {
            //定義參數集合
            List<LogEventProperty> collectedProperties = new List<LogEventProperty>();

            //取得事件或方法(C# Reflection)
            StackTrace stackTrace = new StackTrace();
            string sRequestMethod = stackTrace.GetFrame(2).GetMethod().Name; //往上兩層

            //給定參數
            var properties = collectedProperties.Concat(new[]
           {
                new LogEventProperty("方法名稱", new ScalarValue(sRequestMethod)),
                new LogEventProperty("TraceID", new ScalarValue(_traceID)),
            });

            //紀錄時間
            if (cost > 0)
            {
                properties = properties.Concat(new[]
                {
                    new LogEventProperty("花費時間", new ScalarValue(cost)),
                });
            }

            //訊息格式
            MessageTemplate _messageTemplate = new MessageTemplateParser().Parse(sMessage); //不帶參數，僅顯示訊息

            //write log
            var evt = new LogEvent(DateTimeOffset.Now, logEventLevel, ex, _messageTemplate, properties);
            _logger.Write(evt);
        }

        public void Information(string sMessage)
        {
            WriteLog(LogEventLevel.Information, sMessage);
            //_logger.Information(sMessage);
        }

        public void Error(string sMessage, Exception ex = null) //error通常有Exception
        {
            WriteLog(LogEventLevel.Error, sMessage, ex);
        }

        public void Warning(string sMessage)
        {
            WriteLog(LogEventLevel.Warning, sMessage);
        }

        /// <summary>
        /// 計算花費時間並寫入Log
        /// </summary>
        public void CompleteLog()
        {
            end = Stopwatch.GetTimestamp();
            decimal cost = GetElapsedMilliseconds(start, end);
            WriteLog(LogEventLevel.Debug, $"Total Cost：{cost}", null, cost);
        }

        private decimal GetElapsedMilliseconds(long start, long stop)
        {
            return (stop - start) * 1000 / (decimal)Stopwatch.Frequency; //計算時間(ms)
        }
    }
}
```

修改About頁面，在log結束時間執行CompleteLog方法

![](../.gitbook/assets/image%20%2897%29.png)

```csharp

        protected void Button1_Click(object sender, EventArgs e)
        {
            log.Init(); //初始化：取定TraceID
            try
            {
                log.Information("Serilog Demo");

                throw new Exception("Test Error"); //手動拋出例外
            }
            catch (Exception ex)
            {
                log.Error("Something wrong", ex); //記錄例外log
            }

            log.CompleteLog(); //結束
        }
```

測試：CompelteLog方法成功寫入Total Cost紀錄，後續可依此log檢示各程式或方法執行狀況

![](../.gitbook/assets/image%20%28201%29.png)

例如，查詢About程式中Button1\_Click方法每次花費時間

可以在Seq的Query上下Query語法, 可以得到時間內、每次交易時間 \(此交易時間可以搭配Seq Dashboard中的Alarm功能進行告警\)

```csharp
@Properties['程式名稱'] = 'About' and 方法名稱 = 'Button1_Click' and 花費時間 > 0
```

![](../.gitbook/assets/image%20%2822%29%20%282%29.png)

#### 7. WriteLog改寫\(2020/01/02 Update\)

不想實作WriteLog方法時，也可以選擇用Push參數至LogContext，以達到動態加入參數功能

因此可以將LogHelper改寫成

Enrich.FromLogContext 宣告事件參數使用LogContext

![](../.gitbook/assets/image%20%2881%29.png)

新增AddLogProperties方法，加入需要的參數

寫入Log前先呼叫AddLogProperties加入參數，再呼叫原生log寫入方法即可

![](../.gitbook/assets/image%20%2891%29%20%281%29.png)

![](../.gitbook/assets/image%20%28274%29.png)

完整LogHelper

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Web;

using System.Web.UI;
using Serilog;
using Serilog.Context;
using Serilog.Events;
using Serilog.Parsing;

namespace SerilogDemo
{
    public class LogHelper
    {
        private ILogger _logger;

        private string _traceID = "";

        //Log Start / End
        private long start;
        private long end;

        /// <summary>
        /// 產生TraceID
        /// </summary>
        private void InitTraceID()
        {
            _traceID = Guid.NewGuid().ToString();
        }

        /// <summary>
        /// 初始化Log
        /// </summary>
        public void Init()
        {
            start = Stopwatch.GetTimestamp(); //開始時間
            InitTraceID();
        }

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

        private void AddLogProperties(decimal cost = -1)
        {
            //取得事件或方法(C# Reflection)
            StackTrace stackTrace = new StackTrace();
            string sRequestMethod = stackTrace.GetFrame(2).GetMethod().Name; //往上兩層

            //給定參數
            LogContext.PushProperty("方法名稱", sRequestMethod);
            LogContext.PushProperty("TraceID", _traceID);

            //紀錄時間
            if (cost > 0)
            {
                LogContext.PushProperty("花費時間", cost);
            }
        }

        public void Information(string sMessage)
        {
            AddLogProperties();
            _logger.Information(sMessage);
        }

        public void Error(string sMessage, Exception ex = null) //error通常有Exception
        {
            AddLogProperties();
            _logger.Error(sMessage, ex);
        }

        public void Warning(string sMessage)
        {
            AddLogProperties();
            _logger.Warning(sMessage);
        }

        /// <summary>
        /// 計算花費時間並寫入Log
        /// </summary>
        public void CompleteLog()
        {
            end = Stopwatch.GetTimestamp();
            decimal cost = GetElapsedMilliseconds(start, end);
            AddLogProperties(cost);
            _logger.Debug($"Total Cost：{cost}");
        }

        private decimal GetElapsedMilliseconds(long start, long stop)
        {
            return (stop - start) * 1000 / (decimal)Stopwatch.Frequency; //計算時間(ms)
        }
    }
}
```

#### 8.建立基底類別

若不想每一個頁面都要重新宣告、設定Log物件\(如下圖\)，可以建立基底類別，相關設定在基底類別實作

![](../.gitbook/assets/image%20%28291%29.png)

建立BasePage類別

override OnLoad事件，把log宣告、設定部分寫在此事件中

![](../.gitbook/assets/image%20%2895%29.png)

```csharp
using System;
using System.Web.UI;

namespace SerilogDemo
{
    public class BasePage : Page
    {

        public LogHelper log;

        protected override void OnLoad(EventArgs e)
        {
            if (!IsPostBack)
            {
                log = new LogHelper(this.GetType().Name, this); //this.GetType().Name 取得程式名稱
                Session["log"] = log; //紀錄至Session
            }
            else
            {
                log = (LogHelper)Session["log"];
            }
            base.OnLoad(e);
        }
    }
}
```

About頁面改繼承BasePage \(原為Page\)

LogHelper物件已在BasePage中宣告完畢，因此不需在另外宣告

![](../.gitbook/assets/image%20%2898%29%20%281%29.png)

其它頁面若有log需求，也僅要繼承BasePage即可，例如

![](../.gitbook/assets/image%20%28115%29.png)

如果需要在每個頁面PageLoad / LoadComplete事件中加入Log，可以修改BasePage

針對這兩個件事進行寫入Log動作

![](../.gitbook/assets/image%20%28127%29.png)

測試結果：

第一次進到頁面會有一組Load / LoadComplete紀錄

按下Button會觸發PostBack，所以一樣會有Load / LoadComplete，但中間會插入事件\(例如Button1 Click事件\)

![](../.gitbook/assets/image%20%2836%29.png)

值得注意的是，這麼做會產生一些bug

例如：OnLoad沒有TraceID，主要原因為TraceID是在Button1 Click事件開始時給定的\(Log.Init\)

![](../.gitbook/assets/image%20%28298%29.png)

以定義上來說TraceID是設計給事件用的，而非整個Request，因此需要將LogHelper修改

1. 將原有的TraceID更名為EventTraceID：生命週期為事件內
2. 新增RequestTraceID：生命週期為整個Request



![](../.gitbook/assets/image%20%28142%29.png)

參數名稱新增/修改

![](../.gitbook/assets/image%20%2869%29%20%281%29.png)

BasePage中加入InitRequestTraceID

![](../.gitbook/assets/image%20%28272%29.png)

測試結果：每一個Request log中都有相同的TraceID\(紅\)，事件中亦有自己的獨立TraceID\(藍\)

![](../.gitbook/assets/image%20%28249%29%20%281%29.png)

### Demo Source Code

上述，ASP WebForm搭配使用Serilog 完整Sample Code如下：

[https://github.com/adjackbid/SerilogSample.git](https://github.com/adjackbid/SerilogSample.git)

