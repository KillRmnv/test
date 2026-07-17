### MATCH patterns

_Describe what to find in the graph_

For instance, a pattern can be used to find Emil's friends:

MATCH (ee:Person)-[:KNOWS]-(friends)
WHERE ee.name = 'Emil' RETURN ee, friends

- `MATCH` describes what nodes will be retrieved based upon the pattern.
- `(ee)` is the node reference that will be returned based upon the `WHERE` clause.
- `-[:KNOWS]-` matches the `KNOWS` relationships (in either direction) from `ee`.
- `(friends)` represents the nodes that are Emil's friends.
- `RETURN` returns the node, referenced here by `(ee)`, and the related `(friends)` nodes found.