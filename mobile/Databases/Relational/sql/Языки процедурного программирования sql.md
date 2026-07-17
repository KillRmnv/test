- **`SQL`** — самый простой, пишешь обычный SQL-запрос как тело функции.
    
    `create function get_now() returns timestamp language sql as $$     select now(); $$;`
    
    ⚡ Хорош для простых вещей, но без условий/циклов.
    
- **`plpgsql`** — «родной» процедурный язык PostgreSQL (расширение SQL).  
    Позволяет использовать **переменные, условия, циклы, обработку ошибок**.
    
- **`plpythonu` / `plpython3u`** — писать функции на Python.
    
    `create function py_add(a int, b int) returns int language plpython3u as $$     return a + b $$;`
    
- **`plperl`** — Perl.
    
- **`pltcl`** — Tcl.
    
- **`plv8`** — JavaScript (через движок V8).
    
- **`pllua`** — Lua.
    
- **C** — можно писать функции как расширения на C для максимальной скорости.