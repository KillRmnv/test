Шаблон:

CREATE FUNCTION \[database_name.]function_name (parameters)

RETURNS data_type AS

BEGIN

    SQL statements

    RETURN value

END;

ALTER FUNCTION \[database_name.]function_name (parameters)

RETURNS data_type AS

BEGIN

    SQL statements

    RETURN value

END;

DROP FUNCTION \[database_name.]function_name;








ПРИМЕР, КОТОРЫЙ ВОЗВРАЩАЕТ ЗНАЧЕНИЕ:
CREATE FUNCTION east_or_west (

@long DECIMAL(9,6)

)

RETURNS CHAR(4) AS

BEGIN

[DECLARE] @return_value CHAR(4);

[SET] @return_value = 'same';

    IF (@long > 0.00) SET @return_value = 'east';

    IF (@long < 0.00) SET @return_value = 'west';

    RETURN @return_value

END;






ПРИМЕР,ЧТО ВОЗВРАЩАЕТ ТАБЛИЦУ:

CREATE FUNCTION east_from_long (

@long DECIMAL(9,6)

)

RETURNS TABLE AS

RETURN

SELECT *

FROM city

WHERE city.long > @long;



Postgres:

create function find_printings_by_state_and_type(
state_per varchar(64),
type_per varchar(64)
)
returns table (
	employee_id integer,
	"Издание" integer,
    "Номер издания" integer,
    "Выписан" boolean,
    "Получен" boolean,
	"Название" varchar(64),
	"Тип" varchar(64)
) language plpgsql
as \$$
begin
	if type_per='Газета' then
	return query 
	select t2.*,t1."Название",t1."Тип"
	from find_printings_by_and_state(state_per) t2 join "Издания" t1
	on t1."Тип"='Газета';
	
	elsif type_per='Журнал' then 
	return query 
	select t2.*,t1."Название",t1."Тип"
	from find_printings_by_and_state(state_per) t2 join "Издания" t1
	on t1."Тип"='Журнал'; 

	else return query
	select t2.*,t1."Название",t1."Тип"
	from find_printings_by_and_state(state_per) t2 join "Издания" t1 on t1."Индекс"=t2."Издание";
	end if;
end;
\$$