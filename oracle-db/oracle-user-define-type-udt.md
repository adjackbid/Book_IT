# Oracle User Define Type \(UDT\)

DB端建立類別、測試用Procedure

```sql
DROP TYPE PERSON;
CREATE TYPE PERSON AS OBJECT
(
    NAME VARCHAR2(10),
    AGE NUMBER(10)
);
```

```sql
CREATE OR REPLACE PROCEDURE PTEST(P IN PERSON,O OUT NVARCHAR2) 
IS            
          
BEGIN   
    
    INSERT INTO TEST(A) VALUES(P.NAME);
    COMMIT;
    DBMS_OUTPUT.PUT_LINE(P.NAME);
    O := 'OK';
    
EXCEPTION

When Others Then
    ROLLBACK;
    O := 'ERROR';

END;


```

C\#. Add Class \(IOracleCusomType實作\)

```csharp
public class PersonDto : IOracleCustomType
{

    //[OracleObjectMapping("NAME")]
    [OracleObjectMapping("NAME")]
    public string NAME { get; set; }

    //[OracleObjectMapping("AGE")]
    [OracleObjectMapping("AGE")]
    public decimal AGE { get; set; }


    public void FromCustomObject(OracleConnection con, IntPtr pUdt)
    {
        OracleUdt.SetValue(con, pUdt, "NAME", NAME);
        OracleUdt.SetValue(con, pUdt, "AGE", AGE);
    }

    public void ToCustomObject(OracleConnection con, IntPtr pUdt)
    {
        NAME = (string)OracleUdt.GetValue(con, pUdt, "NAME");
        AGE = (decimal)OracleUdt.GetValue(con, pUdt, "AGE");
    }


    [OracleCustomTypeMapping("PERSON")]
    public class PERSONFactory : IOracleCustomTypeFactory
    {
        //Implementation of interface IOracleCustomTypeFactory method CreateObject.
        //Return a new .NET custom type object representing an Oracle UDT object.
        public IOracleCustomType CreateObject()
        {
            return new PersonDto();
        }
    }
}
```



```csharp
using (OracleConnection conn = new OracleConnection(sConnStr))
{
    conn.Open();
    using (OracleCommand cmd = new OracleCommand("PTEST", conn)) //PTEST = Procedure Name
    {
        cmd.CommandType = System.Data.CommandType.StoredProcedure;

        PersonDto p = new PersonDto() { NAME = "bbb", AGE = 456 };

        OracleParameter p1 = new OracleParameter();
        p1.OracleDbType = OracleDbType.Object;
        p1.Direction = System.Data.ParameterDirection.Input;
        p1.UdtTypeName = "PERSON";
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

