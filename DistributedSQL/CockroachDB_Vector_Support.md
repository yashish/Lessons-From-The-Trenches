CockroachDB supports Vector column as a data type alongside the structured data columns in a table. You can perform ANN (Approximate Nearest Neighbor) vector searches
using C-SPANN algorithm with vector indexes built on cosine (angle) and Euclidean distance similarity in a high-dimensional graph for semantic seraches - not just traditional text searches.
The vector column being alongside the traditional structured data columns ensure strong consistency and ACID compliance where your data lives.

Your vector data (and index) is also fresh because the data lives with it and no need for CDC and keeping vector data consistent with the structured data. 
Updating the vector index without losing quality of the data is always a challenge otherwise.

The idea is you can also create the customer data vector here within a CockroachDB database during the data insert/update using triggers. So the vector data embedding and vectorization is taken care of
by the database and you do not need a pipeline with a python script to embed the data and push it to a separate vector database like Chroma or Pinecone. You also do not need a separate pythin script to 
search the vector data separately as CockroachDb can handle the serach in the same SQL query for the customer data - say searching for similar customers in an e-commerce site for product recommendations.

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
