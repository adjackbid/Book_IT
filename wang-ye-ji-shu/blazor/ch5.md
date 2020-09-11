# CH5



![](../../.gitbook/assets/image%20%28370%29.png)



![](../../.gitbook/assets/image%20%28322%29.png)



![](../../.gitbook/assets/image%20%28317%29.png)



![](../../.gitbook/assets/image%20%28342%29.png)



![](../../.gitbook/assets/image%20%28347%29.png)



![](../../.gitbook/assets/image%20%28307%29.png)



![](../../.gitbook/assets/image%20%28304%29.png)



![](../../.gitbook/assets/image%20%28363%29.png)



![](../../.gitbook/assets/image%20%28331%29.png)



![](../../.gitbook/assets/image%20%28301%29.png)



![](../../.gitbook/assets/image%20%28329%29.png)



![](../../.gitbook/assets/image%20%28373%29.png)

![](../../.gitbook/assets/image%20%28303%29.png)

![](../../.gitbook/assets/image%20%28366%29.png)



![](../../.gitbook/assets/image%20%28335%29.png)

## CH5-2

![](../../.gitbook/assets/image%20%28306%29.png)



![](../../.gitbook/assets/image%20%28321%29.png)



```csharp
using BlazorAppTest.Models;
using Microsoft.EntityFrameworkCore;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Security.Cryptography.X509Certificates;
using System.Threading.Tasks;

namespace BlazorAppTest.Services
{
    public class MyNoteDbService : IMyNoteService
    {
        public MyNoteDbContext _myNoteDbContext { get; }

        public MyNoteDbService(MyNoteDbContext myNoteDbContext)
        {
            _myNoteDbContext = myNoteDbContext;
        }

        public async Task CreateAsync(MyNote myNote)
        {
            await _myNoteDbContext.MyNotes.AddAsync(myNote);
            await _myNoteDbContext.SaveChangesAsync();
        }

        public async Task<List<MyNote>> RetriveAsync()
        {
            return await _myNoteDbContext.MyNotes.ToListAsync();
        }

        public async Task UpdateAsync(MyNote oriNote, MyNote myNote)
        {
            var item = await _myNoteDbContext.MyNotes.FirstOrDefaultAsync(x => x.Id == oriNote.Id);

            if(item != null)
            {
                item.Title = myNote.Title;
                await _myNoteDbContext.SaveChangesAsync();
            }
        }

        public async Task DeleteAsync(MyNote myNote)
        {
            _myNoteDbContext.MyNotes.Remove(await _myNoteDbContext.MyNotes.FirstOrDefaultAsync(x => x.Id == myNote.Id));
            await _myNoteDbContext.SaveChangesAsync();
        }
    }
}
```





![](../../.gitbook/assets/image%20%28357%29.png)





![](../../.gitbook/assets/image%20%28330%29.png)



![](../../.gitbook/assets/image%20%28332%29.png)

