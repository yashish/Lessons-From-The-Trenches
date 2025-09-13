CockroahDB supports Vector column as a data type alongside the structured data columns in a table. You can perform ANN (Approximate Nearest Neighbor) vector searches
using C-SPANN algorithm with vector indexes built on cosine (angle) and Euclidean distance similarity in a high-dimensional graph for semantic seraches - not just traditional text searches.

<-> L2/Euclidean
<=> Cosine

```sql
CREATE TABLE customer (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email STRING NOT NULL,
  date_of_birth DATE NULL,
  location GEOMETRY NULL,
  vec VECTOR(6) NOT NULL,
  VECTOR INDEX (vec)
);

WITH
my_vec AS (
  SELECT vec FROM customer WHERE id = 'abc1234dfg.....'
),
nearest_customers AS (
  SELECT
    c.id as customer_id,
    (c.vec <-> my.vec) AS score
  FROM customer c
  CROSS JOIN my_vec my
  WHERE c.id != 'abc1234dfg.....'
    AND (c.vec <-> my.vec) < 0.5
  ORDER BY c.vec <-> my.vec
  LIMIT 20
)
SELECT * FROM nearest_customers;
```
