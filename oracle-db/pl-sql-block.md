# PL/SQL Block

## Hello World

```sql
SET SERVEROUTPUT ON;

DECLARE
    --宣告變數區
    v_test NVARCHAR2(100) := 'test123';
    v_a INT := 0;
    v_b INT := 100;
    v_c INT;
BEGIN
    --程式碼執行區
    DBMS_OUTPUT.PUT('HELLO ');
    DBMS_OUTPUT.PUT_LINE('WORLD ' || v_test);
    v_c := v_b / v_a;
EXCEPTION
    --異常處理
    --格式：WHEN EXCEPTION_TYPE THEN
    WHEN OTHERS THEN
       DBMS_OUTPUT.PUT_LINE('ERROR[SQLCODE]:' || SQLCODE); --可到oracle官網查表(若為已知)
       DBMS_OUTPUT.PUT_LINE('ERROR[SQLERRM]:' || SQLERRM); --錯誤訊息
END;
```

![](../.gitbook/assets/image%20%2818%29.png)

## Procedure

通常而言，Procedure不會回傳值，可能有傳入值並在執行資料庫的CRUD操作\(insert / update / delete\)

簡例：

```sql
CREATE OR REPLACE PROCEDURE P_HELLO_WORLD
(v_msg IN NVARCHAR2)IS
BEGIN
    DBMS_OUTPUT.put_line(v_msg);
END;
```

呼叫Procedure可以使用Exec或自定義Block後執行整段SQL

![](../.gitbook/assets/image%20%28465%29.png)

![](../.gitbook/assets/image%20%28102%29.png)

## Function

```sql
CREATE OR REPLACE FUNCTION F_SUM_AB(v_a INT, v_b INT)
RETURN INT
IS
BEGIN
    RETURN v_a + v_b;
END;
```

Function可以在選取欄位中使用，例如：

![](../.gitbook/assets/image%20%28476%29.png)

或是WHERE條件中使用

但是需注意，如果A或B是Index，Where條件中使用Function加工過後，Sql Plan就不會使用該Index\(無法使用\)

若真有的需要這樣使用做查詢的話，可以在該Table建立Function Index = F\_SUM\_AB\(A,B\)

![](../.gitbook/assets/image%20%2829%29.png)

如果使用Function時，想要動態取代變數時，可以在變數中加入&

```sql
SELECT F_SUM_AB(&A,&B)
  FROM DUAL
```

執行時，就會跳出變數輸入對話框，決定A、B變數

![](../.gitbook/assets/image%20%28469%29.png)

輸入完後，就會帶入該變數

![](../.gitbook/assets/image%20%28377%29.png)

