create trigger [trigger_name] 
[before | after]  
{insert | update | delete}  
on [table_name]  
FOR EACH ROW
BEGIN
END;


CREATE TRIGGER [trigger_name] 
ON DATABASE
FOR CREATE_TABLE, ALTER_TABLE, DROP_TABLE
AS 
BEGIN
  
   ROLLBACK;
END;


CREATE TRIGGER [trigger_name]  
ON  [table_name] 
FOR UPDATE 
AS 
BEGIN 

   ROLLBACK; 
END;


CREATE TRIGGER track_logon
ON LOGON
AS
BEGIN
END;