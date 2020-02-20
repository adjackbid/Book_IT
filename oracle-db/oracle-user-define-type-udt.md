# Oracle User Define Type \(UDT\)

```sql
create type Person as object
(
    Person_Name NVARCHAR2(10),
    Age integer
);
```



![](../.gitbook/assets/image%20%28268%29.png)



```sql
CREATE OR REPLACE FUNCTION FUNTEST(P IN PERSON)
RETURN nvarchar2 IS

BEGIN

   RETURN P.PERSON_NAME;

   EXCEPTION
   When Others Then
   RETURN 'error...';
END; 
/
```

```sql
SELECT FUNTEST(PERSON('JACK',10)) AS TEST FROM DUAL;
```

![](../.gitbook/assets/image%20%28474%29.png)





```sql
CREATE TYPE PERSON_LIST AS TABLE OF PERSON;
```

![](../.gitbook/assets/image%20%28398%29.png)



```sql
CREATE OR REPLACE FUNCTION FUNTEST2(PS IN PERSON_LIST)
RETURN nvarchar2 IS

BEGIN

   RETURN PS(2).PERSON_NAME;

   EXCEPTION
   When Others Then
   RETURN 'error...';
END;
/

```

```sql
SELECT FUNTEST2(PERSON_LIST(PERSON('A',1),PERSON('B',2))) AS TEST2
  FROM DUAL
```

![](../.gitbook/assets/image%20%28382%29.png)





