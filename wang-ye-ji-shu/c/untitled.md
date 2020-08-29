---
description: Expression Tree
---

# Untitled

Func

Func可以被Invoke

```csharp
    public class DemoClass
    {
        public bool MyProperty { get; set; }

        public string DoSomething(int number, string text) => (number + text);

    }
    
```

```csharp
            var demo = new DemoClass();

            //delegate
            Func<DemoClass, string> func = c => c.DoSomething(123, "test");

            var result = func(demo);

            Console.WriteLine(result);
```

![](../../.gitbook/assets/image%20%28295%29.png)



![](../../.gitbook/assets/image%20%28287%29.png)



expression

```csharp
Expression<Func<DemoClass, string>> expr = c => c.DoSomething(42, "test");
```

不能被Invoke



List 中使用func案例

![](../../.gitbook/assets/image%20%28284%29.png)

LIST中每一個元件會INVOKE FUNC \(t=&gt; t.Length &gt; 0\) 並回傳string

使用expression tres 案例

![](../../.gitbook/assets/image%20%28294%29.png)

expression tree是一種表示式，執行時 entity framework會透過expression tree特性

取得表示式中的body及每一個元素，例如t、 t.Length 、&gt;、 0

並將其轉換成對應的SQL

表式示\(樹狀結構\)表達一個方法或CLASS

![](../../.gitbook/assets/image%20%28279%29.png)



![](../../.gitbook/assets/image%20%28292%29.png)

![](../../.gitbook/assets/image%20%28286%29.png)



![](../../.gitbook/assets/image%20%28282%29.png)



![](../../.gitbook/assets/image%20%28290%29.png)

