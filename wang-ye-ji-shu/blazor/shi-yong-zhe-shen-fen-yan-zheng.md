# 使用者身分驗證

安裝Microsoft.AspNetCore.Blazor.HttpClient

![](../../.gitbook/assets/image%20%28383%29.png)

新增Login.cshtml

![](../../.gitbook/assets/image%20%28385%29.png)



設計頁面

![](../../.gitbook/assets/image%20%28386%29.png)

調整Login.cshtml.cs

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

#region 這裡將會是新加入的命名空間宣告
using System.Security.Claims;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authorization;
#endregion


namespace BlazorAppTest.Pages
{
    [AllowAnonymous]
    public class LoginModel : PageModel
    {
        public string ReturnUrl { get; set; }
        public async Task<IActionResult> OnGetAsync(string paramUsername, string paramPassword)
        {
            string returnUrl = Url.Content("~/");
            try
            {
                // 清除已經存在的登入 Cookie 內容
                await HttpContext
                    .SignOutAsync(
                    CookieAuthenticationDefaults.AuthenticationScheme);
            }
            catch { }

            #region 這裡將會要針對傳入的使用者帳號與密碼進行驗證

            #region 本練習簡化將不做任何驗證，不過，本練習將簡化不做任何設計

            #endregion

            #region 加入這個使用者需要用到的 宣告類型 Claim Type
            var claims = new List<Claim>
            {
                new Claim(ClaimTypes.Name, paramUsername),
                new Claim(ClaimTypes.Role, "Administrator"),
            };
            #endregion

            #region 建立 宣告式身分識別
            // ClaimsIdentity類別是宣告式身分識別的具體執行, 也就是宣告集合所描述的身分識別
            var claimsIdentity = new ClaimsIdentity(
                claims, CookieAuthenticationDefaults.AuthenticationScheme);
            #endregion

            #region 建立關於認證階段需要儲存的狀態
            var authProperties = new AuthenticationProperties
            {
                IsPersistent = true,
                RedirectUri = this.Request.Host.Value
            };
            #endregion

            #region 進行使用登入
            try
            {
                await HttpContext.SignInAsync(
                CookieAuthenticationDefaults.AuthenticationScheme,
                new ClaimsPrincipal(claimsIdentity),
                authProperties);
            }
            catch (Exception ex)
            {
                string error = ex.Message;
            }
            #endregion

            #endregion

            return LocalRedirect(returnUrl);
        }
    }
}

```

新增Logout

![](../../.gitbook/assets/image%20%28396%29.png)

![](../../.gitbook/assets/image%20%28389%29.png)



調整Logout.cshtml.cs

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

#region 這裡將會是新加入的命名空間宣告
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
#endregion

namespace BlazorAppTest.Pages
{
    public class LogoutModel : PageModel
    {
        public async Task<IActionResult> OnGetAsync()
        {
            // 清除已經存在的登入 Cookie 內容
            await HttpContext
                .SignOutAsync(
                CookieAuthenticationDefaults.AuthenticationScheme);
            return LocalRedirect(Url.Content("~/"));
        }
    }
}

```



在Shared中新增razor元件

![](../../.gitbook/assets/image%20%28391%29.png)

```csharp
@page "/LoginControl"
<AuthorizeView>
    <Authorized>
        <b>Hello, @context.User.Identity.Name!</b>
        <a class="ml-md-auto btn btn-primary"
           href="/logout?returnUrl=/"
           target="_top"> 登出 </a>
    </Authorized>
    <NotAuthorized>
        <input type="text"
               placeholder="User Name"
               @bind="@Username" />
        &nbsp;&nbsp;
        <input type="password"
               placeholder="Password"
               @bind="@Password" />
        <a class="ml-md-auto btn btn-primary"
           href="/login?paramUsername=@Username&paramPassword=@Password"
           target="_top"> 登入 </a>
    </NotAuthorized>
</AuthorizeView>
@code {
    string Username = "";
    string Password = "";
}
```

設定Start up

![](../../.gitbook/assets/image%20%28388%29.png)

![](../../.gitbook/assets/image%20%28393%29.png)

![](../../.gitbook/assets/image%20%28395%29.png)



![](../../.gitbook/assets/image%20%28390%29.png)

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.HttpsPolicy;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using BlazorAppTest.Data;
using BlazorAppTest.Services;
using BlazorAppTest.Models;
using Microsoft.EntityFrameworkCore;
using Blazored.Modal;
using System.Net.Http.Headers;

#region 這裡將會是新加入的命名空間宣告
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Http;
using System.Net.Http;
#endregion

namespace BlazorAppTest
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
        public void ConfigureServices(IServiceCollection services)
        {

            #region 加入使用 Cookie 認證需要的宣告
            services.Configure<CookiePolicyOptions>(options =>
            {
                options.CheckConsentNeeded = context => true;
                options.MinimumSameSitePolicy = Microsoft.AspNetCore.Http.SameSiteMode.None;
            });
            services.AddAuthentication(
                CookieAuthenticationDefaults.AuthenticationScheme)
                .AddCookie();
            #endregion


            services.AddRazorPages();
            services.AddServerSideBlazor();
            services.AddSingleton<WeatherForecastService>();

            //DI
            //services.AddScoped<IMyNoteService, MyNoteService>();
            //services.AddScoped<IMyNoteService, MyNoteDbService>();
            //services.AddScoped<IMyNoteService, MyNoteWebAPIService>();

            //宣告使用SQL Lite資料庫
            services.AddDbContext<MyNoteDbContext>(options =>
            {
                options.UseSqlite("Data Source=MyNote.db");
            });

            services.AddBlazoredModal(); //調用BlazoredModal

            //新增Control、API相關功能支援，但不需要VIEW、PAGES
            services.AddControllers();

            //使用IHttpClientFactory註冊IMyNoteService服
            services.AddHttpClient<IMyNoteService, MyNoteWebAPIService>(c =>
            {
                c.BaseAddress = new Uri("https://localhost:44361");
                //預設使用json做為預設請求
                c.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            });

            #region 加入會用到的服務宣告
            services.AddHttpContextAccessor();
            services.AddScoped<HttpContextAccessor>();
            services.AddHttpClient();
            services.AddScoped<HttpClient>();
            #endregion

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

            #region 指定要使用 Cookie & 使用者認證的中介軟體
            app.UseCookiePolicy();
            app.UseAuthentication();
            #endregion

            app.UseRouting();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers(); //新增屬性路由控制器的支援
                endpoints.MapBlazorHub();
                endpoints.MapFallbackToPage("/_Host");
            });
        }
    }
}

```

修改app.razor, 加入

```csharp
<CascadingAuthenticationState>
```

```csharp
<CascadingAuthenticationState>
    <CascadingBlazoredModal>
        <Router AppAssembly="@typeof(Program).Assembly">
            <Found Context="routeData">
                <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
            </Found>
            <NotFound>
                <LayoutView Layout="@typeof(MainLayout)">
                    <p>Sorry, there's nothing at this address.</p>
                </LayoutView>
            </NotFound>
        </Router>
    </CascadingBlazoredModal>
</CascadingAuthenticationState>
```

修改MainLayout.razor

```csharp
@inherits LayoutComponentBase

@*<BlazoredModal/>*@

<div class="sidebar">
    <NavMenu />
</div>

<div class="main">
    <div class="top-row px-4">
        @*在這裡加入剛剛建立的登出入 Blazor 元件*@
        <LoginControl />
        <a href="https://docs.microsoft.com/aspnet/" target="_blank">About</a>
    </div>

    <div class="content px-4">
        @Body
    </div>
</div>

```

修改Index

![](../../.gitbook/assets/image%20%28387%29.png)

測試，程式邏輯：



* 請執行這個 Blazor 專案
* 將會看到如下圖的網頁顯示出來
* 在網頁的最上方，將會看到剛剛設計的 \[LoginControl.razor\] 元件
* 因為現在尚未通過身分驗證，所以，在 \[LoginControl.razor\] 元件上將會看到登入按鈕
* 而且，在首頁上因為使用了 `AuthorizeView` 這個 Blazor 內建的元件，因此，將會根據使用者是否有成功通過驗證，而顯示出不同的內容
* 由於現在尚未通過驗證，因此，在首頁上將會顯示尚未通過驗證的訊息![](https://vulcanlee.gitbooks.io/csharp2019/content/Images/CSharp9911.png)
* 請在 \[User Name\] 與 \[Password\] 欄位，輸入使用的帳號與密碼 \(因為在這個練習專案中，並未實際做使用帳號與密碼的檢查，因此，可以輸入任何內容即可\)
* 請點選右上方的 \[登入\] 按鈕，如下面螢幕截圖![](https://vulcanlee.gitbooks.io/csharp2019/content/Images/CSharp9910.png)
* 在成功通過身分驗證過程，也就是登入成功了，在網頁最上方的 \[LoginControl.razor\] 元件內容也自動切換成已經登入的狀態
* 並且，在首頁頁面上，也因為成功登入了，顯示出已經登入訊息，這裡的內容將會動態的切換，與尚未成功登入前有所不同![](https://vulcanlee.gitbooks.io/csharp2019/content/Images/CSharp9909.png)
* 因為採用 Cookie 的方式，所以，不論關閉這個網頁，或者開啟一個新的瀏覽器標籤頁次，都會看到如上面螢幕截圖的畫面

