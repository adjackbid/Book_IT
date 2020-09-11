---
description: 在 Blazor 專案加入 Web API 服務的功能
---

# CH7

![](../../.gitbook/assets/image%20%28318%29.png)



define json prop name

![](../../.gitbook/assets/image%20%28312%29.png)



![](../../.gitbook/assets/image%20%28328%29.png)



![](../../.gitbook/assets/image%20%28342%29.png)



![](../../.gitbook/assets/image%20%28380%29.png)

controller

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using BlazorAppTest.Models;
using System.Security.Cryptography.X509Certificates;

namespace BlazorAppTest.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class MyNotesController : ControllerBase
    {
        public MyNoteDbContext _myNoteDbContext { get; }

        public MyNotesController(MyNoteDbContext myNoteDbContext)
        {
            _myNoteDbContext = myNoteDbContext;
        }

        [HttpGet]
        public IEnumerable<MyNote> Get()
        {
            return _myNoteDbContext.MyNotes.ToList();
        }

        [HttpGet("{id}")]
        public MyNote Get(int Id)
        {
            return _myNoteDbContext.MyNotes.FirstOrDefault(x => x.Id == Id);
        }

        [HttpPost]
        public void Post([FromBody] MyNote note)
        {
            _myNoteDbContext.MyNotes.Add(note);
        }

        [HttpPut("{id}")]
        public void Put(int id, [FromBody] MyNote note)
        {
            var item = _myNoteDbContext.MyNotes.FirstOrDefault(x => x.Id == note.Id);
            item.Title = note.Title;
            _myNoteDbContext.SaveChanges();
        }

        [HttpDelete("{id}")]
        public void Delete(int id)
        {
            _myNoteDbContext.MyNotes.Remove(_myNoteDbContext.MyNotes.FirstOrDefault(x => x.Id == id));
            _myNoteDbContext.SaveChanges();
        }

    }
}

```

test

![](../../.gitbook/assets/image%20%28355%29.png)

