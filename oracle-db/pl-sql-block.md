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
       DBMS_OUTPUT.PUT_LINE('ERROR[SQLCODE]:' || SQLCODE);
       DBMS_OUTPUT.PUT_LINE('ERROR[SQLERRM]:' || SQLERRM);
END;
```

![](../.gitbook/assets/image%20%2818%29.png)



