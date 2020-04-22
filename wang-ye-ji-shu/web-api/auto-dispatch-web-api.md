# Auto Dispatch Web API

## 需求

1.API Json需統一格式為：

```csharp
{
 "FunctionName":"", //方法名稱
 "FunctionUID":"", //交易id
 "FunctionType":"",//S：送出；R：回傳
 "Content": //該方法使用的參數
 {
   //依目標方法決定需要參數、型態
 }
}
```

2. 呼叫端統一API入口：[http://xxx.xxx.xx/API/](http://xxx.xxx.xx/API/)

由API自行依照FunctionName Dispath方法

Example： 傳入

```csharp
{
    "FunctionName":"DoSomething", //方法名稱
    "FunctionUID":"1", //交易id
    "FunctionType":"S",//S：送出；R：回傳
    "Content": //該方法使用的參數
    {
        "A":"123"
    }
}
```

回傳

```csharp
{
    "FunctionName":"DoSomething", //方法名稱
    "FunctionUID":"1", //交易id
    "FunctionType":"R",//S：送出；R：回傳
    "Content": //該方法使用的參數
    {
        "B":"456",
        "C":"789"
    }
}
```

## 實作\(C\#\)

建立公版API格式類別 APIData

```csharp
 public class APIData
 {
     //固定格式
     public string FunctionName { get; set; }
     public string FunctionUID { get; set; } //uni id
     public string FunctionType { get; set; } //S：Send / R：Return

     //先定義成object
     public object Content { get; set; } //內容

     public string returncode { get; set; } //00

     public string returnmessage { get; set; } //OK
 }
```

建立Dosomething 傳入、回傳Content類別

```csharp
 //傳入
 public class DoSomthingContentInput
 {
     public string A { get; set; }
 }

 //回傳
 public class DoSomethingContentResult
 {
     public string B { get; set; }
     public string C { get; set; }
 }
```

建立統一API入口 假設目標方法位於同一個class內\(非必要\) 利用reflection找到方法並呼叫該方法

```csharp
 [HttpPost]
 public object Post(APIData JsonObj) //Request DataBinding to APIDataObj
 {
     //Get Target Method
     var func = this.GetType().GetMethod(JsonObj.FunctionName, BindingFlags.NonPublic | BindingFlags.Instance);
     if (func == null)
     {
         JsonObj.returncode = "01";
         JsonObj.returnmessage = "Function Name Not Found!!";
         JsonObj.FunctionType = "R";
         return JsonObj;
     }
     return func.Invoke(this, new object[] { JsonObj}); //呼叫方法
 }
```

建立DoSomething方法 將收到的apidata中content轉成該方法使用的content input class 處理完畢後，回傳結果

```csharp
 private object DoSomething(APIData JsonObj)
 {
     //change to DoSomethingContentInput obj
     var content_input = JsonConvert.DeserializeObject<DoSomthingContentInput>(JsonObj.Content?.ToString());

     //Do Something
     var content_result = new DoSomethingContentResult()
     {
         B = $"{content_input.A}",
         C = "CCC",
     };

     JsonObj.Content = content_result;
     JsonObj.FunctionType = "R";
     JsonObj.returncode = "00";
     JsonObj.returnmessage = "OK";
     return JsonObj;
 }
```

測試\(PostMan\)：

![](../../.gitbook/assets/image%20%28318%29.png)

完整程式碼：[github](https://github.com/adjackbid/DispatchWebAPI)

