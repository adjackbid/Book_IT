# Var 隱含型別

```csharp
int x = 123;
var y = 123;
```

var y為隱含型別，編輯器於編譯時期就會決定型別，因此使用var宣告時，一定要給初始值

var不能使用在function傳入值使用 \(因為無法判斷型別\)



![](../../.gitbook/assets/image%20%28261%29.png)

```csharp
int? x = null;
int? y = x ?? 1;  ==> int? y = (x!= null) ? x : 1;
```

