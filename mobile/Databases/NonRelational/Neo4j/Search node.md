### MATCH

_Find nodes_

Now, find the node representing Emil.

1. Click this code block and bring it into the Editor:
    
    MATCH (ee:Person) WHERE ee.name = 'Emil' RETURN ee;
    
    - `MATCH` specifies a pattern of nodes and relationships.
    - `(ee:Person)` is a single node pattern with label `Person`. It assigns matches to the variable `ee`.
    - `WHERE` filters the query.
    - `ee.name = 'Emil'` compares name property to the value `Emil`.
    - `RETURN` returns particular results.