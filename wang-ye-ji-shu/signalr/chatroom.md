---
description: ASP.Net Core搭配SignalR示範ChatRoom功能
---

# ChatRoom

## 專案建立

建立ASP.Net Core Web應用程式

![](../../.gitbook/assets/image%20%28140%29.png)

![](../../.gitbook/assets/image%20%2853%29.png)

選擇Web應用程式

![](../../.gitbook/assets/image%20%2870%29.png)

## 新增用戶端程式庫

點選專案→加入→用戶端程式庫

![](../../.gitbook/assets/image%20%288%29.png)

提供者選擇unpkg

程式庫輸入：@microsoft/signalr@latest

安裝 =&gt; 將signalr.js安裝至專案根目錄wwwroot/lib/@microsoft/signalr中

![](../../.gitbook/assets/image%20%28258%29.png)

![](../../.gitbook/assets/image%20%28180%29.png)

## Hub建立

專案中建立資料夾Hubs

建立ChatHub類別

![](../../.gitbook/assets/image%20%28215%29.png)

```csharp
using Microsoft.AspNetCore.SignalR;
using System.Threading.Tasks;

namespace ChatRoom.Hubs
{
    public class ChatHub : Hub
    {
        public async Task SendMessage(string user, string message)
        {
            await Clients.All.SendAsync("ReceiveMessage", user, message);
        }
    }
}
```

## 前端UI

Index頁面，新增2個input\(text\)、1個button、訊息列\(messagelist\)

![](../../.gitbook/assets/image%20%28188%29.png)

```aspnet
@page
<div class="container">
    <div class="row">&nbsp;</div>
    <div class="row">
        <div class="col-2">User</div>
        <div class="col-4"><input type="text" id="userInput" /></div>
    </div>
    <div class="row">
        <div class="col-2">Message</div>
        <div class="col-4"><input type="text" id="messageInput" /></div>
    </div>
    <div class="row">&nbsp;</div>
    <div class="row">
        <div class="col-6">
            <input type="button" id="sendButton" value="Send Message" />
        </div>
    </div>
</div>
<div class="row">
    <div class="col-12">
        <hr />
    </div>
</div>
<div class="row">
    <div class="col-6">
        <ul id="messagesList"></ul>
    </div>
</div>
<script src="~/js/signalr.js"></script>
<script src="~/js/chat.js"></script>
```

新增chat.js \(~/js/目錄下\)

![](../../.gitbook/assets/image%20%28242%29.png)

chat.js

```csharp
"use strict";

var connection = new signalR.HubConnectionBuilder().withUrl("/chatHub").build();

//Disable send button until connection is established
document.getElementById("sendButton").disabled = true;

connection.on("ReceiveMessage", function (user, message) {
    var msg = message.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
    var encodedMsg = user + " says " + msg;
    var li = document.createElement("li");
    li.textContent = encodedMsg;
    document.getElementById("messagesList").appendChild(li);
});

connection.start().then(function () {
    document.getElementById("sendButton").disabled = false;
}).catch(function (err) {
    return console.error(err.toString());
});

document.getElementById("sendButton").addEventListener("click", function (event) {
    var user = document.getElementById("userInput").value;
    var message = document.getElementById("messageInput").value;
    connection.invoke("SendMessage", user, message).catch(function (err) {
        return console.error(err.toString());
    });
    event.preventDefault();
});
```

將signalr.js copy至js資料下 \(從lib\@microsoft\signalr\dist\browser\)

![](../../.gitbook/assets/image%20%28153%29.png)

## 服務DI注入

Startup.cs中將SignalR注入 \(ConfigureService Method中\)

並設定Endpoint

![](../../.gitbook/assets/image%20%2895%29.png)

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.HttpsPolicy;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using ChatRoom.Hubs;

namespace ChatRoom
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddRazorPages();
            services.AddSignalR();
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Error");
                // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
                app.UseHsts();
            }

            app.UseHttpsRedirection();
            app.UseStaticFiles();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapRazorPages();
                endpoints.MapHub<ChatHub>("/chatHub");//setting hub
            });
        }
    }
}
```

## 簡易測試

Debug Mode，開啟兩個視窗即可測試SignalR即時回應功能

![](../../.gitbook/assets/image%20%28293%29.png)

以上參考微軟官方Demo範例\([https://docs.microsoft.com/zh-tw/aspnet/core/tutorials/signalr?view=aspnetcore-3.0&tabs=visual-studio](https://docs.microsoft.com/zh-tw/aspnet/core/tutorials/signalr?view=aspnetcore-3.0&tabs=visual-studio)\)

## 功能修改 - 新增連線功能及線上人數

UI調整\(Index.cshtml\) 新增連線Button、目前人數Label

![](../../.gitbook/assets/image%20%28148%29.png)

```aspnet
@page
    <div class="container">
        <div class="row">&nbsp;</div>
        <div class="row">
            <div class="col-2">User</div>
            <div class="col-4"><input type="text" id="userInput" /></div>
        </div>
        <div class="row">
            <div class="col-2">Message</div>
            <div class="col-4"><input type="text" id="messageInput" /></div>
        </div>
        <div class="row">&nbsp;</div>
        <div class="row">
            <div class="col-6">
                <input type="button" id="connectToHub" value="Connect" />
                <input type="button" id="sendButton" value="Send Message" />
            </div>
        </div>

        <div class="row">
            <div class="col-6">
                <label>Current：</label>
                <label id="UserCount">0</label>
            </div>
        </div>

    </div>
<div class="row">
    <div class="col-12">
        <hr />
    </div>
</div>
<div class="row">
    <div class="col-6">
        <ul id="messagesList"></ul>
    </div>
</div>
<script src="~/js/signalr.js"></script>
<script src="~/js/chat.js"></script>
```

chat.js修改

connectToHub物件加入Click EvenListener：按下連線Button後，才進行連線、調整計數器

![](../../.gitbook/assets/image%20%2816%29.png)

chat.js

```csharp
"use strict";

var connection; 

document.getElementById("sendButton").disabled = true;

document.getElementById("sendButton").addEventListener("click", function (event) {
    
    var user = document.getElementById("userInput").value;

    var message = document.getElementById("messageInput").value;

    if(message == "")
    {
        alert("message不可為空!!");
        return;
    }

    connection.invoke("SendMessage", user, message).catch(function (err) {
        return console.error(err.toString());
    });
    event.preventDefault();
});

document.getElementById("connectToHub").addEventListener("click", function (event) {
    
    var user = document.getElementById("userInput").value;
   
    if(user == "")
    {
        alert("user不可為空!!");
        return;
    }

    //?username目的為傳入使用者名稱至後端
    connection = new signalR.HubConnectionBuilder().withUrl("/chatHub?username=" + user).build();

    document.getElementById("connectToHub").disabled = true;
    
    connection.on("ReceiveMessage", function (user, message) {
        var msg = message.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
        var encodedMsg = user + "：" + msg;
        var li = document.createElement("li");
        li.textContent = encodedMsg;
        document.getElementById("messagesList").appendChild(li);
    });
    
    connection.on("PushMessage", function (message) {
        var msg = message.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
        var encodedMsg = "Sysem：" + msg;
        var li = document.createElement("li");
        li.textContent = encodedMsg;
        document.getElementById("messagesList").appendChild(li);
    });

    //UserCount
    connection.on("SetUserCount", function (UserCount) {
        document.getElementById("UserCount").innerHTML = UserCount;
    });

    connection.start().then(function () {
        document.getElementById("sendButton").disabled = false;
    }).catch(function (err) {
        return console.error(err.toString());
    });

    event.preventDefault();
});
```

修改ChatHub.cs 修改連線、斷線非同步方法

![](../../.gitbook/assets/image%20%284%29.png)

```csharp
using Microsoft.AspNetCore.SignalR;
using System.Threading.Tasks;
using System;

namespace ChatRoom.Hubs
{
    public class ChatHub : Hub
    {
        public static int UsersCount = 0;
        private string Now => DateTime.Now.ToString("HH:mm:ss");

        public async Task SendMessage(string user, string message)
        {
            await Clients.All.SendAsync("ReceiveMessage", user, message);
        }

        //連線
        public override async Task OnConnectedAsync()
        {
            UsersCount++;
            var username = Context.GetHttpContext().Request.Query["username"]; //取得使用者名稱
            //await Clients.All.SendAsync("ReceiveMessage", "System", $"[{Now}] {username} is connected");
            await Clients.All.SendAsync("PushMessage", $"[{Now}] {username} is connected"); //推送訊息
            await Clients.All.SendAsync("SetUserCount", UsersCount); //更新計數器
        }

        //離線
        public override async Task OnDisconnectedAsync(Exception ex)
        {
            UsersCount--;
            var username = Context.GetHttpContext().Request.Query["username"]; //取得使用者名稱
            await Clients.All.SendAsync("PushMessage", $"[{Now}] {username} left"); //推送訊息
            await Clients.All.SendAsync("SetUserCount", UsersCount); //更新計數器
        }
    }
}
```

## 測試

註：測試時，若chat.js沒有重新載入修改後的版本\(因為有catch\)，可按Ctrl+F5即會重新載入js檔

![](../../.gitbook/assets/image%20%28159%29.png)

