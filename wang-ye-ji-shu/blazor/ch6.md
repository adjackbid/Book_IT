---
description: 在 Blazor 專案內呼叫 JavaScript 程式碼 1
---

# CH6



![](../../.gitbook/assets/image%20%28313%29.png)



![](../../.gitbook/assets/image%20%28310%29.png)

\_host.cshtml add js

```text
@page "/"
@namespace BlazorAppTest.Pages
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@{
    Layout = null;
}

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>BlazorAppTest</title>
    <base href="~/" />
    <link rel="stylesheet" href="css/bootstrap/bootstrap.min.css" />
    <link href="css/site.css" rel="stylesheet" />
</head>
<body>
    <app>
        <component type="typeof(App)" render-mode="ServerPrerendered" />
    </app>

    <div id="blazor-error-ui">
        <environment include="Staging,Production">
            An error has occurred. This application may no longer respond until reloaded.
        </environment>
        <environment include="Development">
            An unhandled exception has occurred. See browser dev tools for details.
        </environment>
        <a href="" class="reload">Reload</a>
        <a class="dismiss">🗙</a>
    </div>

    <script src="_framework/blazor.server.js"></script>

    <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js">
    </script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js"></script>

    <script>
        function ShowModal(modalId) {
            $('#' + modalId).modal('show');
        }
        function CloseModal(modalId) {
            $('#' + modalId).modal('hide');
        }
    </script>

</body>
</html>

```



![](../../.gitbook/assets/image%20%28366%29.png)



![](../../.gitbook/assets/image%20%28360%29.png)



![](../../.gitbook/assets/image%20%28335%29.png)



![](../../.gitbook/assets/image%20%28375%29.png)

注入JSRuntime\(執行JS用\)

調整ShowModal寫法

```csharp
@*透過route方式*@
@page "/note"
@using BlazorAppTest.Models
@using BlazorAppTest.Services;
@**inject，當BLAZOR元件建立後，注意IMyNoteService抽象型別對應DI型別並給定至NoteService*@
@inject IMyNoteService NoteService
@*注入此物件，使C#可以呼叫JavaScript*@
@inject IJSRuntime jsRuntime
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
        @foreach (var note in Notes)
        {
            <tr>
                <td>@note.Title</td>
                <td><input type="button" class="btn btn-primary" value="Edit" @onclick="()=>Update(note)" /></td>
                <td><input type="button" class="btn btn-danger" value="Delete" @onclick="()=>Delete(note)" /></td>
            </tr>
        }
    </tbody>
</table>

<div>
    @*Blazor資駃定，將新增按鈕的點選事件綁定至C#之delegate method*@
    <input type="button" class="btn btn-primary" value="新增" @onclick="()=>Add()" />
</div>

@*Shop pop windows*@
<div class="modal" tabindex="-1" role="dialog" id="@DialogIdName">
    <div class="modal-dialog" role="document">
        <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title">Title</h5>
                <button type="button" class="close" @onclick="()=>CloseDialog()">
                    <span aria-hidden="true">x</span>
                </button>
            </div>
            <div class="modal-body">
                <div class="row">
                    <div class="col-9">
                        <div class="row p-2">
                            @*Blazor 提供的EditForm元件進行輸入表單驗證*@
                            @*Edit form提供有效資料驗證 OnValidSubmit事件*@
                            <EditForm Model="@CurrentNote" OnValidSubmit="@HandleValidSubmit">
                                @*DataAnnotationsValidator 元件會使用資料模型的屬性宣造，將驗證支援附加至EditContext*@
                                @*ValidationSummary元件會匯整所有驗證訊息*@
                                <DataAnnotationsValidator />
                                <ValidationSummary />
                                <div class="form-group">
                                    <label for="name">Title</label>
                                    <InputText id="name" class="form-control" @bind-Value="@CurrentNote.Title">

                                    </InputText>
                                </div>
                                <button type="submit" class="btn btn-primary">Save</button>
                            </EditForm>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

@code {
    public List<MyNote>
    Notes { get; set; } = new List<MyNote>();


    public bool ShowPopup = false;

    public MyNote CurrentNote { get; set; } = new MyNote();
    public MyNote OriNote { get; set; } = new MyNote();

    public bool IsAdd { get; set; }

    public string DialogIdName { get; set; } = "myModal";

    //protected override void OnInitialized()
    //{
    //    Notes = new List<MyNote>()
    //    {
    //        new MyNote{Title="A"},
    //        new MyNote{Title="B"}
    //    };
    //}

    protected override async Task OnInitializedAsync()
    {
        Notes = await NoteService.RetriveAsync();
    }

    private async void Add()
    {
        //Notes.Add(new MyNote { Title = $"New Item {DateTime.Now.ToString()}" }); // for test
        IsAdd = true;
        CurrentNote = new MyNote();
        //ShowPopup = true;
        //Js function name & Value
        await jsRuntime.InvokeAsync<object>("ShowModal", DialogIdName);
    }

    //private void Delete(MyNote note)
    //{
    //    Notes.Remove(note);
    //}

    private async void Delete(MyNote note)
    {
        await NoteService.DeleteAsync(note);

        //更新資料後，重整
        Notes = await NoteService.RetriveAsync();
        StateHasChanged();//不同執行緒時，變更物件值時，可通知BLAZOR 檢查畫面中是否有變更，若有變更更新至UI上

    }

    private async void Update(MyNote note)
    {
        //update
        IsAdd = false;
        CurrentNote = OriNote = note.Clone(); //避免修改後 沒按SAVE，但舊資料被修改過
                                              //產生一個物件副本，做為更新用
        CurrentNote = note.Clone();
        OriNote = note;
        //ShowPopup = true;
        await jsRuntime.InvokeAsync<object>("ShowModal", DialogIdName);
    }

    private async void CloseDialog()
    {
        //ShowPopup = false;
        await jsRuntime.InvokeAsync<object>("CloseModal", DialogIdName);
    }

    //private void HandleValidSubmit()
    //{
    //    CloseDialog();
    //    if(IsAdd)
    //    {
    //        Notes.Add(CurrentNote);
    //    }
    //    else
    //    {
    //        OriNote.Title = CurrentNote.Title;
    //    }
    //}

    private async void HandleValidSubmit()
    {
        CloseDialog();
        if(IsAdd)
        {
            await NoteService.CreateAsync(CurrentNote);
        }
        else
        {
            await NoteService.UpdateAsync(OriNote, CurrentNote);
        }

        Notes = await NoteService.RetriveAsync();
        StateHasChanged();//不同執行緒時，變更物件值時，可通知BLAZOR 檢查畫面中是否有變更，若有變更更新至UI上
    }

            }


```

![](../../.gitbook/assets/image%20%28315%29.png)



## CH6-2



![](../../.gitbook/assets/image%20%28351%29.png)

![](../../.gitbook/assets/image%20%28352%29.png)

![](../../.gitbook/assets/image%20%28374%29.png)

![](../../.gitbook/assets/image%20%28368%29.png)

using，減少重覆USING，在import中加入

![](../../.gitbook/assets/image%20%28316%29.png)



POP MODAL放在app.razor中，\(每一頁都可以使用\)

![](../../.gitbook/assets/image%20%28367%29.png)

新增CSS USING

![](../../.gitbook/assets/image%20%28373%29.png)

using Js

```csharp
@page "/"
@namespace BlazorAppTest.Pages
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@{
    Layout = null;
}

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>BlazorAppTest</title>
    <base href="~/" />
    <link rel="stylesheet" href="css/bootstrap/bootstrap.min.css" />
    <link href="css/site.css" rel="stylesheet" />
    <link href="~/_content/Blazored.Modal/blazored-modal.css" rel="stylesheet" />
</head>
<body>
    <app>
        <component type="typeof(App)" render-mode="ServerPrerendered" />
    </app>

    <div id="blazor-error-ui">
        <environment include="Staging,Production">
            An error has occurred. This application may no longer respond until reloaded.
        </environment>
        <environment include="Development">
            An unhandled exception has occurred. See browser dev tools for details.
        </environment>
        <a href="" class="reload">Reload</a>
        <a class="dismiss">🗙</a>
    </div>

    <script src="_framework/blazor.server.js"></script>

    <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js">
    </script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js"></script>
    <script src="~/_content/Blazored.Modal/blazored.modal.js"></script>
    <script>
        function ShowModal(modalId) {
            $('#' + modalId).modal('show');
        }
        function CloseModal(modalId) {
            $('#' + modalId).modal('hide');
        }
    </script>

</body>
</html>

```

![](../../.gitbook/assets/image%20%28341%29.png)

```csharp
@inject IModalService ModalService

<div class="simple-form">
    <div class="form-group">
        <p>Are your sure to delete this @RecordTitleName data? </p>
    </div>

    <button @onclick="SubmitForm" class="btn btn-danger">Delete</button>
    <button @onclick="Cancel" class="btn btn-secondary">Cancel</button>
</div>

@code {
    //CascadingParameter = Blazor元件參數使用方式
    //[CascadingParameter] ModalParameters _parameters { get; set; }
    //[CascadingParameter] BlazoredModal _blzaoredModal { get; set; }
    [CascadingParameter] BlazoredModalInstance _blazoredModal { get; set; }

    [Parameter]public string RecordTitleName { get; set; } //RecordTitleName

    protected override void OnInitialized()
    {
        //RecordTitleName = _parameters.Get<string>("RecordTitleName");
    }

    void SubmitForm()
    {
        _blazoredModal.Close(ModalResult.Ok($"RecordTitleName {RecordTitleName} was deleted"));
        //ModalService.Show(ModalResult.Ok($"RecordTitleName {RecordTitleName} was deleted"));
    }

    void Cancel()
    {
        //ModalService.Show(ModalResult.Cancel());
        _blazoredModal.Cancel();
    }

}

```



![](../../.gitbook/assets/image%20%28350%29.png)



Mynote.Razor改寫Delete部分 \([https://github.com/Blazored/Modal](https://github.com/Blazored/Modal)\)

```csharp
private async Task Delete(MyNote note)
    {
        CurrentNote = note;
        var parameters = new ModalParameters();
        //傳參數
        parameters.Add("RecordTitleName", note.Title);

        var formModal = modal.Show<ConfirmDelete>("Are u sure?", parameters);

        var result = await formModal.Result;

        if(result.Cancelled)
        {
            Console.WriteLine("Model as cancelled");
        }
        else
        {
            Console.WriteLine(result.Data);
            await DeleteIt(CurrentNote);
            StateHasChanged();
        }
    }

    private async Task DeleteIt(MyNote note)
    {
        await NoteService.DeleteAsync(note);
        Notes = await NoteService.RetriveAsync();
    }
```



![](../../.gitbook/assets/image%20%28327%29.png)

