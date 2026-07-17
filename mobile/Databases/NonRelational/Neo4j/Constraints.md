### Create constraints

_Unique node property constraints_

Create unique node property constraints to ensure that property values are unique for all nodes with a specific label. Adding the unique constraint, implicitly adds an index on that property.

CREATE CONSTRAINT FOR (n:Movie) REQUIRE (n.title) IS UNIQUE

CREATE CONSTRAINT FOR (n:Person) REQUIRE (n.name) IS UNIQUE