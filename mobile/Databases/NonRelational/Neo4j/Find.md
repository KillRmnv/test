### Find

_Find individual nodes_

1. Run any of the following query examples.
2. Notice the syntax pattern.
3. Try looking for other movies or actors.

---

- Find the actor named "Tom Hanks":
    
    MATCH (tom:Person {name: "Tom Hanks"}) RETURN tom
    
- Find the movie with title "Cloud Atlas":
    
    MATCH (cloudAtlas:Movie {title: "Cloud Atlas"}) RETURN cloudAtlas
    
- Find 10 people and return their names:
    
    MATCH (people:Person) RETURN people.name LIMIT 10
    
- Find movies released in the 1990s and return their titles.
    
    MATCH (nineties:Movie) WHERE nineties.released >= 1990 AND nineties.released < 2000 RETURN nineties.title