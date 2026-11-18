# Week Task 5: Vector Search Internals – Cosine vs Euclidean

## Objective

Understand vector database internals by implementing brute-force vector search using NumPy and comparing the results with Qdrant.

The project demonstrates:

1. Euclidean distance calculation.
2. Cosine similarity calculation.
3. Brute-force Top-K search using NumPy.
4. Exact nearest-neighbour search using Qdrant.
5. Vector normalisation and dot-product equivalence.
6. Performance comparison between NumPy and Qdrant.
7. HNSW indexing concepts.

## Technologies

- Python
- NumPy
- Pandas
- Matplotlib
- Qdrant Cloud
- Qdrant Python Client

## Dataset

The benchmark uses 10,000 reproducible synthetic vectors with 512 dimensions.

The same dataset is used for both Euclidean and cosine searches.

Dataset sizes tested:
- 1,000 vectors
- 10,000 vectors

## 1. Euclidean Distance

Euclidean distance measures the straight-line distance between two vectors.

Formula:

d(x,y) = sqrt(sum((x_i-y_i)^2))

The smaller the distance, the more similar the vectors are according to Euclidean distance.

## 2. Cosine Similarity

Cosine similarity measures the angle between two vectors.

Formula:

cos(x,y) = (x . y) / (||x|| * ||y||)

A higher cosine similarity indicates a more similar direction.

## 3. NumPy Brute-Force Search

NumPy calculates distances or similarities between the query vector and every vector in the dataset.

The Top-K results are selected using sorting.

Euclidean:
- Calculate distances.
- Sort in ascending order.
- Return the smallest K distances.

Cosine:
- Calculate similarities.
- Sort in descending order.
- Return the largest K similarities.

## 4. Qdrant Search

Two Qdrant collections are created:

- task5_euclidean
- task5_cosine

The same vectors are uploaded into both collections.

Exact search is enabled using SearchParams(exact=True).

The returned vector IDs are compared with NumPy results to verify correctness.

## 5. Vector Normalisation

A vector is normalised by dividing it by its magnitude.

x_normalised = x / ||x||

For normalised vectors, the magnitude becomes 1.

Therefore:

Cosine Similarity = Dot Product

Both methods produce the same ranking when vectors are normalised.

## 6. Performance Benchmark

The search performance is tested using:

- 1,000 vectors
- 10,000 vectors

Both NumPy and Qdrant execute Euclidean and cosine searches.

The median of five measured search runs is used.

Actual benchmark results:

| Vector Count | NumPy (ms) | Qdrant (ms) |
|---|---|---|
| 1,000 | Fill actual result | Fill actual result |
| 10,000 | Fill actual result | Fill actual result |

Benchmark measures query latency and excludes initial vector upload time.

## 7. What is HNSW?

HNSW stands for Hierarchical Navigable Small World.

It is a graph-based approximate nearest-neighbour search algorithm.

Instead of comparing a query against every vector, HNSW navigates through a graph of connected vectors to find likely nearest neighbours.

Advantages:
- Faster search on large datasets.
- Efficient nearest-neighbour retrieval.
- Scalable vector search.

Trade-off:
- Approximate search may not always return the mathematically exact nearest neighbours.
- Index construction requires additional memory and processing.

Our NumPy and Qdrant exact-search comparison verifies exact results. The benchmark does not claim to measure approximate HNSW search.

## Conclusion

This project demonstrates how vector similarity search works internally.

NumPy provides a brute-force baseline, while Qdrant provides a vector database implementation.

The comparison verifies that both systems return matching nearest-neighbour IDs when exact search is used.

Vector normalisation demonstrates why cosine similarity and dot product produce the same ranking for unit vectors.

The benchmark compares query latency for different dataset sizes and introduces the role of HNSW in scalable vector databases.
