### CREATE

_Create a node_

Let's use Cypher to generate a small social graph.

**NOTE:** This guide assumes that you use an empty graph.

1. Click this code block and bring it into the Editor:
    
    CREATE (ee:Person {name: 'Emil', from: 'Sweden', kloutScore: 99})
    
    - `CREATE` creates the node.
    - `()` indicates the node.
    - `ee:Person` – `ee` is the node variable and`Person` is the node label.
    - `{}` contains the properties that describe the node.