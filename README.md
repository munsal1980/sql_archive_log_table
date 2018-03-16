CREATE OR REPLACE FUNCTION "ECZANE"."CREATELOGINSERTSTATEMENT" (
   pSourceOwner          IN VARCHAR2,
   pSourceTable          IN VARCHAR2,
   pSourceColumnPrefix   IN VARCHAR2,
   pTargetOwner          IN VARCHAR2,
   --pTargetTable             VARCHAR2,
   pLogId                IN NUMBER,
   pLogAck               IN VARCHAR2)
   --pLogUsr               IN VARCHAR2)
   RETURN VARCHAR2
IS
pDate VARCHAR2(25):='SYSDATE';
pUsr  VARCHAR2(25):='USER';
   CURSOR stmt_cur
   IS
      SELECT 'INSERT INTO ' || pTargetOwner || '.Z_' || pSourceTable || 
      ' ('|| RTRIM (
                   EXTRACT (
                      XMLAGG (
                         XMLELEMENT (
                            "a",
                            column_name || ',')),
                      '//text()'),
                   ',')
             || ', LOGID , LOGACK ,LOGTRH,LOGUSR)'||
             ' VALUES('
             || RTRIM (
                   EXTRACT (
                      XMLAGG (
                         XMLELEMENT (
                            "a",
                            pSourceColumnPrefix || '.' || column_name || ',')),
                      '//text()'),
                   ',')
             || ',' || pLogId || ',''' || pLogAck || '''' ||   ','|| pDate || '' ||  ',' ||pUsr|| ');'
                AS inserttatement
        FROM all_tab_columns
       WHERE owner = pSourceOwner AND table_name = pSourceTable ORDER BY COLUMN_ID;
BEGIN
   FOR v_rec IN stmt_cur
   LOOP
      RETURN v_rec.inserttatement;
   END LOOP;
END;
