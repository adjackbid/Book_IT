---
description: 使用 Web API 來儲存記事服務相關紀錄
---

# CH8

![](../../.gitbook/assets/image%20%28334%29.png)



建立MyNoteWebAPIService

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using BlazorAppTest.Models;
using System.Text.Json;
using System.Text;
using System.Net.Http;
using System.Threading;

namespace BlazorAppTest.Services
{
    public class MyNoteWebAPIService : IMyNoteService
    {
        public HttpClient Client { get; }

        public MyNoteWebAPIService(HttpClient httpClient)
        {
            Client = httpClient;
        }
        


        public async Task CreateAsync(MyNote myNote)
        {
            var content = JsonSerializer.Serialize(myNote);
            using (var stringContent = new StringContent(content, Encoding.UTF8, "application/json")) 
            {
                await Client.PostAsync("/api/MyNotes", stringContent);
            }
        }

        public async Task DeleteAsync(MyNote myNote)
        {
            await Client.DeleteAsync($"/api/MyNotes/{myNote.Id}");
        }

        public async Task<List<MyNote>> RetriveAsync()
        {
            var content = await Client.GetStringAsync("/api/MyNotes");
            var items = JsonSerializer.Deserialize<List<MyNote>>(content);
            return items;
        }

        public async Task UpdateAsync(MyNote oriNote, MyNote myNote)
        {
            var content = JsonSerializer.Serialize(myNote);
            using(var stringContent = new StringContent(content,Encoding.UTF8,"application/json"))
            {
                await Client.PutAsync($"/api/MyNotes/{myNote.Id}", stringContent);
            }
        }
    }
}

```



Startup 調整Configure Service

![](../../.gitbook/assets/image%20%28363%29.png)



![](../../.gitbook/assets/image%20%28356%29.png)



