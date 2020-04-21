# DataTypes

## 文字型態

**CHAR**：宣告固定長度的字元型態，宣告後，不論變數所存的字元數，**所使用的長度仍為宣告時定義的長度**

**VARCHAR2**：宣告變動長度的字元型態，宣告時定義最大的長度，使用的長度依實際資料儲存的長度動態決定

**NCHAR**：National Characters and Strings；支援全球語系的unicode編碼，使用長度計算方式同CHAR

**NVARCHAR2**：National Characters and Strings；支援全球語系的unicode編碼，使用長度計算方式同VARCHAR2

NCHAR、NVARCHAR2的最大長度可到32,767bytes，**但是會因為儲存的字元之編碼而有所不同**

例如：

UTF8編碼時，每一個字元佔3 bytes，所以最多可儲存10,922 \(32,767 / 3\)個字元

AL16UTF16編碼時，每一個字元佔2 bytes，所以最多可儲存16,383 \(32,767 / 2\)個字元

在使用國際字元時，通常會在字元前加一個N，例如

```sql
v_test NVARCHAR2(100) := N'測試';
```

註：不加也不會發生什麼事…

![](../.gitbook/assets/image%20%28275%29.png)

## Composite DataTypes

複合變數的主要特性為可以包含一個以上的值，主要的資料型態有RECORD、COLLECTION兩種類型

### RECORD

類似C\#的Structure，可以在其中定義多個欄位的一種型態，例如：

```sql
DECLARE
    --定義RECORD型態
    TYPE REC_TEST IS RECORD
    (
        A NVARCHAR2(10),
        B NVARCHAR2(20)
    );
    
    --宣告一個變數且型別為REC_TEST
    v_test REC_TEST;
BEGIN

    v_test.A := 'a123';
    v_test.B := 'b456';
    
    DBMS_OUTPUT.PUT_LINE(v_test.A || v_test.B);

END;
```

![](../.gitbook/assets/image%20%28443%29.png)

### COLLECTION

Collection為一個有序的集合，每一個項目都有相同的資料型態

Oracle支援的Collection型態共有三種：

1. VARRAY Datatype
2. Nested Table Datatype
3. Associative Array Datatype\(Index-by Tables\)

