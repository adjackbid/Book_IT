# DataTypes

## 文字型態

**CHAR**：宣告固定長度的字元型態，宣告後，不論變數所存的字元數，所使用的長度仍為宣告時定義的長度

**VARCHAR2**：宣告變動長度的字元型態，宣告時定義最大的長度，使用的長度依實際資料儲存的長度動態決定

**NCHAR**：National Characters and Strings；支援全球語系的unicode編碼，使用長度計算方式同CHAR

**NVARCHAR2**：National Characters and Strings；支援全球語系的unicode編碼，使用長度計算方式同VARCHAR2

NCHAR、NVARCHAR2的最大長度可到32,767bytes，但是會因為儲存的字元之編碼而有所不同

例如：

UTF8編碼時，每一個字元佔3 bytes，所以最多可儲存10,922 \(32,767 / 3\)個字元

AL16UTF16編碼時，每一個字元佔2 bytes，所以最多可儲存16,383 \(32,767 / 2\)個字元

在使用國際字元時，通常會在字元前加一個N，例如

```sql
v_test NVARCHAR2(100) := N'測試';
```

註：不加也不會發生什麼事…

![](../.gitbook/assets/image%20%28266%29.png)



