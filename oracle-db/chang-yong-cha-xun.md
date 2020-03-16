# 常用查詢

## 1. Group By String \(Concat\)

遇到需要將文字透過Group By方式接在一個欄位時，可以使用listagg

```sql

SELECT listagg(COL1, ',') within group (order by COL1) as ALL_COL1 
  FROM
  (
    SELECT 'A' AS COL1,'A1' AS COL2
      FROM DUAL
      UNION ALL
    SELECT 'A' AS COL1,'A2' AS COL2
      FROM DUAL
      UNION ALL
    SELECT 'B' AS COL1,'B1' AS COL2
      FROM DUAL
      UNION ALL
    SELECT 'B' AS COL1,'B2' AS COL2
      FROM DUAL
  ) GROUP BY COL2
 
-- ORACLE 11g => WM_CONCAT
```



