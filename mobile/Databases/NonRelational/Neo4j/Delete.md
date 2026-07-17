### Clean up

_Remove the movie data set_

When you are done experimenting, you can clean up your graph.

**NOTE:** Nodes cannot be deleted if they have relationships, so you need to detach the nodes to delete them.

---

1. Delete all `Movie` and `Person` nodes, and their relationships.
    
    MATCH (n) DETACH DELETE n
    
2. Verify that the Movie Graph has been removed.
    
    MATCH (n) RETURN n



Для того, чтобы удалить элемент из БД используется оператор 

MATCH ... [WHERE ...] DELETE ...


Примечание: в neo4j невозможно удалить вершину, если она связана с другими какими-либо отношениями.

  
Сначала нужно удалить связи этой вершины с другими:

MATCH (n: Person) OPTIONAL MATCH (n)-[r]-(m) DELETE r

  

Оператор OPTIONAL MATCH означает, что, если какая-либо пара вершин не связана искомым отношением, но удовлетворяет остальным требованиям, она будет отмечена.

**Затем удалить вершины:

MATCH (n: Person) DELETE n

  

Или удалить всё сразу:

MATCH (n: Person) OPTIONAL MATCH (n)-[r]-(m) DELETE n, r, m

