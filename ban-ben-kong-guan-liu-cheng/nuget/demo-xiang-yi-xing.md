# Demo-相依性

## 相依性情境

假設內部有兩個Nuget套件：

* LogTool\(負責處理Log Handling\)
* DBTool\(負責處理DB層資料CRUD\)

![](../../.gitbook/assets/image%20%28217%29.png)

LogTool在寫入Log時，需要寫入DB，因此會使用到DBTools

## Nuget Package新增相依性

在Nuget Package Explorer中，編輯Package metadata

![](../../.gitbook/assets/image%20%28195%29.png)

點選Edit dependencies

![](../../.gitbook/assets/image%20%28140%29.png)

新增Group

![](../../.gitbook/assets/image%20%28151%29.png)

若相依的套件，有發佈在Nuget Server上\(私有或公有都可\)，即可點選下方按紐選擇套件

![](../../.gitbook/assets/image%20%28212%29.png)

Package Source選擇自架的Nuget Server

點選相依的Nuget套件

![](../../.gitbook/assets/image%20%28203%29.png)

加入後，按下ok即可

![](../../.gitbook/assets/image%20%28238%29.png)

存檔後，即可看到metadata中會顯示Dependencies

![](../../.gitbook/assets/image%20%2876%29.png)

後續專案在安裝LogTool套件時，則會自動連同相依的DBTools套件一併安裝

![](../../.gitbook/assets/image%20%28165%29.png)

![](../../.gitbook/assets/image%20%28107%29.png)

![](../../.gitbook/assets/image%20%28210%29.png)

