### Query

_Find patterns_

Use the type of the relationship to find patterns within the graph, for example, `ACTED_IN` or `DIRECTED`. What other relationships exist?

---

- What movies did Tom Hanks act in?
    
    MATCH (tom:Person {name: "Tom Hanks"})-[:ACTED_IN]->(tomHanksMovies) RETURN tom,tomHanksMovies
    
- Who directed "Cloud Atlas"?
    
    MATCH (cloudAtlas:Movie {title: "Cloud Atlas"})<-[:DIRECTED]-(directors) RETURN directors.name
    
- Who were Tom Hanks' co-actors?
    
    MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors) RETURN DISTINCT coActors.name
    
- How people are related to "Cloud Atlas"?
    
    MATCH (people:Person)-[relatedTo]-(:Movie {title: "Cloud Atlas"}) RETURN people.name, Type(relatedTo), relatedTo.roles
    
    
    
    ### Query product data

_Query patterns_

Lets try some queries using patterns.

![](http://localhost:7474/browser/assets/images/northwind/product-graph.png)

---

- What categories of food does each supplier supply?
    
    MATCH (s:Supplier)-->(:Product)-->(c:Category)
    RETURN s.companyName as Company, collect(distinct c.categoryName) as Categories
    
- Find the produce suppliers.
    
    MATCH (c:Category {categoryName:"Produce"})<--(:Product)<--(s:Supplier)
    RETURN DISTINCT s.companyName as ProduceSuppliers
    
    
    
**Если при поиске какой-либо промежуточный элемент не представляет интереса, можно оставить в шаблоне пустые скобки: () - для вершины, [] - для связи.**
**Если искомые вершины могут быть соединены отношением любого типа, то в шаблоне поиска такое отношение обозначается -[*]-**

**

Если необходимо найти какое-то определённое количество узлов, после оператора RETURN необходимо использовать оператор LIMIT amount, где amount - количество необходимых вершин.

Так же можно отсортировать найденные данные c помощью оператора ORDER BY (ASC - прямой порядок сортировки, DESC - обратный).**