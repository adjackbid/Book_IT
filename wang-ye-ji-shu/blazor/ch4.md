# CH4

![](../../.gitbook/assets/image%20%28346%29.png)



![](../../.gitbook/assets/image%20%28365%29.png)

![](../../.gitbook/assets/image%20%28364%29.png)

```csharp
@code {
    public List<MyNote> Notes { get; set; } = new List<MyNote>();

    public bool ShowPopup = false;

    public MyNote CurrentNote { get; set; } = new MyNote();
    public MyNote OriNote { get; set; } = new MyNote();

    public bool IsAdd { get; set; }

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

    private void Add()
    {
        //Notes.Add(new MyNote { Title = $"New Item {DateTime.Now.ToString()}" }); // for test
        IsAdd = true;
        CurrentNote = new MyNote();
        ShowPopup = true;
    }

    //private void Delete(MyNote note)
    //{
    //    Notes.Remove(note);
    //}

    private async void Delete(MyNote note)
    {
        await NoteService.DeleteAsync(note);
    }

    private void Update(MyNote note)
    {
        //update
        IsAdd = false;
        CurrentNote = OriNote = note.Clone(); //避免修改後 沒按SAVE，但舊資料被修改過
        //產生一個物件副本，做為更新用
        CurrentNote = note.Clone();
        OriNote = note;
        ShowPopup = true;
    }

    private void CloseDialog()
    {
        ShowPopup = false;
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

