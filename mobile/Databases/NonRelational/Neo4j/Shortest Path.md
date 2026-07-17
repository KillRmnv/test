- Use variable length patterns to find movies and actors up to 4 "hops" away from Kevin Bacon.
    
    MATCH (bacon:Person {name:"Kevin Bacon"})-[*1..4]-(hollywood)
    RETURN DISTINCT hollywood
    
- Use the built-in `shortestPath()` algorithm to find the "Bacon Path" to Meg Ryan.
    
    MATCH p=shortestPath(
    (bacon:Person {name:"Kevin Bacon"})-[*]-(meg:Person {name:"Meg Ryan"})
    )
    RETURN p