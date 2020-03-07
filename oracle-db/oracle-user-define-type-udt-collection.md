# Oracle User Define Type \(UDT\) - Collection

C\# 呼叫Oracle Function / Procedure時，如果要使用DB上的自定義類似且為Array\(多筆\)時，可參考以下Demo：

測試用Table

```sql
CREATE TABLE TEST
(
  A  NVARCHAR2(10),
  B  NVARCHAR2(10),
  C  NVARCHAR2(10)
)
TABLESPACE SYSTEM
RESULT_CACHE (MODE DEFAULT)
PCTUSED    40
PCTFREE    10
INITRANS   1
MAXTRANS   255
STORAGE    (
            INITIAL          64K
            NEXT             1M
            MINEXTENTS       1
            MAXEXTENTS       UNLIMITED
            PCTINCREASE      0
            FREELISTS        1
            FREELIST GROUPS  1
            BUFFER_POOL      DEFAULT
            FLASH_CACHE      DEFAULT
            CELL_FLASH_CACHE DEFAULT
           )
LOGGING 
NOCOMPRESS 
NOCACHE
NOPARALLEL
MONITORING;
```

自定類別PERSON

```sql
CREATE OR REPLACE TYPE PERSON as object
(
    NAME VARCHAR2(10),
    AGE NUMBER(2)
);
/
```

自定集合型別PERSON\_LIST為PERSON的集合

```sql
CREATE OR REPLACE TYPE PERSON_LIST AS TABLE OF PERSON;
```

測試用Procedure，傳入PERSON\_LIST\(多筆PERSON\)並寫入TABLE TEST

```sql
CREATE OR REPLACE PROCEDURE PTEST2(P IN PERSON_LIST,O OUT NVARCHAR2) 
IS            
          
BEGIN   
    
    IF P IS NULL THEN
        RAISE_APPLICATION_ERROR(-20000, 'PERSON_LIST is null!');
    END IF;
    
    FOR I IN P.FIRST .. P.LAST
    LOOP
        INSERT INTO  TEST(A)
        VALUES (P(I).NAME);
    END LOOP;

    COMMIT;
    O := 'OK';
    
EXCEPTION

When Others Then
    ROLLBACK;
    O := 'ERROR';
END;
```

C\#：使用自定義的Oracle Collection型別，同樣要實作IOracleCustomType

```csharp
public class PersonListDto : IOracleCustomType
{
    [OracleArrayMapping()]
    public PersonDto[] PersonDtos;

    public void FromCustomObject(OracleConnection con, IntPtr pUdt)
    {
        OracleUdt.SetValue(con, pUdt, 0, PersonDtos);
    }

    public void ToCustomObject(OracleConnection con, IntPtr pUdt)
    {
        PersonDtos = (PersonDto[])OracleUdt.GetValue(con, pUdt, 0);
    }

    [OracleCustomTypeMapping("PERSON_LIST")]
    public class PersonListFactory : IOracleCustomTypeFactory, IOracleArrayTypeFactory
    {
        public IOracleCustomType CreateObject()
        {
            return new PersonListDto();
        }

        public Array CreateArray(int numElems)
        {
            return new PersonDto[numElems];
        }

        public Array CreateStatusArray(int numElems)
        {
            return null;
        }
    }
}
```

傳入參數OracleDbType需選擇Array

```csharp
using (OracleConnection conn = new OracleConnection(sConnStr))
{
    conn.Open();
    using (OracleCommand cmd = new OracleCommand("PTEST2", conn))
    {
        cmd.CommandType = System.Data.CommandType.StoredProcedure;
    
        PersonListDto p = new PersonListDto();
        
        //兩筆測試資料
        PersonDto[] p_array = new PersonDto[] { new PersonDto() {NAME="a",AGE=1 },
                                                new PersonDto() { NAME = "b",AGE=2 } };
        p.PersonDtos = p_array;
    
        OracleParameter p1 = new OracleParameter();
        p1.OracleDbType = OracleDbType.Array;
        p1.Direction = System.Data.ParameterDirection.Input;
        p1.UdtTypeName = "PERSON_LIST";
        p1.Value = p;
        p1.IsNullable = false;
        p1.ParameterName = "P";
        cmd.Parameters.Add(p1);
    
        OracleParameter pr = new OracleParameter();
        pr.OracleDbType = OracleDbType.NVarchar2;
        pr.Direction = System.Data.ParameterDirection.Output;
        pr.Size = 100;
        cmd.Parameters.Add(pr);
        cmd.ExecuteNonQuery();
        sResult = (string)(OracleString)cmd.Parameters[1].Value;
    }
}
```

測試結果：成功寫入兩筆資料

![](../.gitbook/assets/image%20%2843%29.png)

