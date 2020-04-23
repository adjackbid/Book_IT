# CH2

![](../../.gitbook/assets/image%20%28444%29.png)

![](../../.gitbook/assets/image%20%28125%29.png)



![](../../.gitbook/assets/image%20%28333%29.png)

![](../../.gitbook/assets/image%20%28185%29.png)



![](../../.gitbook/assets/image%20%28477%29.png)

```csharp
<h3>MyNotes</h3>

@*Hmtl*@
<table class="table">
    <thead>
        <tr>
            <th>事項</th>
            <th>修改</th>
            <th>刪除</th>
        </tr>
    </thead>
    <tbody>

    </tbody>
</table>

<div>
    @*Blazor資駃定，將新增按鈕的點選事件綁定至C#之delegate method*@
    <input type="button" class="btn btn-primary" value="新增" />
 </div>

@code {

}

```

![](../../.gitbook/assets/image%20%28232%29.png)

![](../../.gitbook/assets/image%20%28392%29.png)

![](../../.gitbook/assets/image%20%28324%29.png)



![](../../.gitbook/assets/image%20%28319%29.png)



![](../../.gitbook/assets/image%20%28153%29.png)



![](../../.gitbook/assets/image%20%28462%29.png)

## 2-2

![](../../.gitbook/assets/image%20%28470%29.png)

```aspnet
@*透過route方式*@
@page "/note"
@using BlazorAppTest.Models

<h3>MyNotes</h3>

@*Hmtl*@
<table class="table">
    <thead>
        <tr>
            <th>事項</th>
            <th>修改</th>
            <th>刪除</th>
        </tr>
    </thead>
    <tbody>
        @*for each item (Display)*@
        @foreach(var note in Notes)
        {
        <tr>
            <td>@note.Title</td>
            <td><input type="button" class="btn btn-primary" value="Edit" /></td>
            <td><input type="button" class="btn btn-danger" value="Delete" @onclick="()=>Delete(note)" /></td>
        </tr>
        }
    </tbody>
</table>

<div>
    @*Blazor資駃定，將新增按鈕的點選事件綁定至C#之delegate method*@
    <input type="button" class="btn btn-primary" value="新增" @onclick="()=>Add()" />
</div>

@code {
    public List<MyNote> Notes { get; set; } = new List<MyNote>();

    protected override void OnInitialized()
    {
        Notes = new List<MyNote>()
        {
            new MyNote{Title="A"},
            new MyNote{Title="B"}
        };
    }

    private void Add()
    {
        Notes.Add(new MyNote { Title = $"New Item {DateTime.Now.ToString()}" }); // for test
    }

    private void Delete(MyNote note)
    {
        Notes.Remove(note);
    }
}


```

![](../../.gitbook/assets/image%20%28381%29.png)



