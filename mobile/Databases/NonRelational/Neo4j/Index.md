### Index nodes

_Create indexes on one or more properties for all nodes that have a given label. Indexes are used to increase search performance._

CREATE INDEX FOR (m:Movie) ON (m.released)