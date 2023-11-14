# ORA-20000: Unable to set values for index xxx: does not exist or insufficient privileges

参考文章：https://www.cnblogs.com/kerrycode/p/17527407.html

问题背景：Oracle19c版本数据导出后导入另一数据库报错，报错信息如标题所示

>Bug 30978304的详细描述如下所示：
>
>Description ORA-20000 error was occurring with Data Pump importing statistics, with certain combinations of indexes. This has now been fixed.   
>
>What Happens?   
>
>After importing a transportable tablespace, extents belonging to an index are incorrectly marked as unallocated in the tablespace bitmaps. This leads to objd mismatch asserts because such extents could eventually be allocated to another object.   
>
>Conditions   
>
>The export-side must have the following properties:   There is a user-created table with a multi-column PK constraint in the table DDL. This constraint has a system-generated unique index (say ABC). There is a user-created unique index (say XYZ) on the same columns as the PK, but the column ordering differs. Note that if the column ordering matches, XYZ creation would have failed with ORA-1408 and this bug wont occur.   
>
>Fix   
>
>The fix forces the creation of ABC earlier. This resolves the corruption   
>
>REDISCOVERY INFORMATION:   
>
>ORA-20000 occurring while impdp is importing statistics, when two or more indexes exist on a table and one of them is a primary key index.   
>
>Additional symptoms:   
>
>TTS import from a 12.1 DB to 19c corrupts the Tablespace bitmaps which can result in the following errors being raised for operations on segments belonging to the Tablespace :   
>
>1. ORA-8103 
>2. ORA-600 [kcl_mismatch_1] 
>3. ORA-600 [kdifind:kcbz_objdchk] 
>4. ORA-600 [ktrget2:kcbz_objdchk] 
>5. ORA-600 [ktspffbmb:objdchk_kcbnew_3] 
>6. ORA-600 [ktspgtb2:kcbz_objdchk]   
>
>Workaround 
>
>None.   
>
>You can likely get this fix in: 
>
>Data Pump Recommended Proactive Patches For 19.10 and Above (Doc ID 2819284.1)

主要意思是，没有对应的解决方案，但是在更高的版本中解决了，这个问题，建议升级到19.10或者更高版本。问题出现的原因是，在一张表的ddl中，出现了大于等于两个索引且有一个是主键索引，然后会出现如标题所示的问题。