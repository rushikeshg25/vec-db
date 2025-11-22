# Vector Database (vec-db)

A comprehensive vector database implementation from scratch in Python. This project provides a complete solution for storing, indexing, and searching high-dimensional vectors with support for multiple indexing strategies and similarity metrics.

## Features

- **Multiple Indexing Strategies**: HNSW (Hierarchical Navigable Small World) for approximate search and brute force for exact search
- **Various Similarity Metrics**: Cosine similarity, Euclidean distance, Dot product, Manhattan distance
- **Metadata Support**: Store and filter by metadata alongside vectors
- **Persistence**: Save and load database state to/from disk
- **Batch Operations**: Efficient bulk insert and search operations
- **Thread-Safe**: Safe for concurrent operations
- **Flexible API**: Simple and intuitive interface

## Installation

```bash
pip install -r requirements.txt
```

Or install as a package:

```bash
pip install -e .
```

## Quick Start

```python
from vec_db import VectorDB
import numpy as np

# Create a vector database
db = VectorDB(dimension=128, distance_metric="cosine")

# Add vectors
vector1 = np.random.rand(128).astype(np.float32)
db.add(id="vec1", vector=vector1, metadata={"category": "A", "score": 0.95})

vector2 = np.random.rand(128).astype(np.float32)
db.add(id="vec2", vector=vector2, metadata={"category": "B", "score": 0.87})

# Search for similar vectors
query_vector = np.random.rand(128).astype(np.float32)
results = db.search(query_vector, k=5)

for result in results:
    print(f"ID: {result['id']}, Distance: {result['distance']}, Metadata: {result['metadata']}")

# Save database
db.save("my_database.pkl")

# Load database
db_loaded = VectorDB.load("my_database.pkl")
```

## Architecture

### Core Components

1. **VectorDB** (`vec_db/core.py`): Main database class managing vectors, metadata, and indexes
2. **Indexes** (`vec_db/indexes/`): Different indexing strategies for similarity search
   - `HNSWIndex`: Fast approximate nearest neighbor search
   - `BruteForceIndex`: Exact search for smaller datasets
3. **Metrics** (`vec_db/metrics.py`): Distance and similarity calculation functions
4. **Persistence** (`vec_db/persistence.py`): Save/load functionality
5. **Query** (`vec_db/query.py`): Query builder and filtering capabilities

### Indexing Strategies

#### HNSW (Hierarchical Navigable Small World)

- **Use Case**: Large datasets requiring fast approximate search
- **Pros**: Very fast, scales well with dataset size
- **Cons**: Approximate results (may miss some nearest neighbors)
- **Parameters**:
  - `M`: Number of bi-directional links (default: 16)
  - `ef_construction`: Size of dynamic candidate list (default: 200)
  - `ef_search`: Size of candidate list during search (default: 50)

#### Brute Force

- **Use Case**: Small datasets or when exact results are required
- **Pros**: Guaranteed exact results
- **Cons**: Slower for large datasets (O(n) complexity)
- **Best For**: Datasets with < 10,000 vectors

## API Reference

### VectorDB Class

#### Initialization

```python
VectorDB(
    dimension: int,
    distance_metric: str = "cosine",  # "cosine", "euclidean", "dot", "manhattan"
    index_type: str = "hnsw",  # "hnsw" or "brute_force"
    **index_kwargs
)
```

#### Methods

- `add(id: str, vector: np.ndarray, metadata: dict = None) -> None`

  - Add a single vector to the database

- `add_batch(vectors: List[Dict]) -> None`

  - Bulk insert vectors. Format: `[{"id": "vec1", "vector": np.array, "metadata": {...}}, ...]`

- `get(id: str) -> Dict`

  - Retrieve a vector by ID

- `update(id: str, vector: np.ndarray = None, metadata: dict = None) -> None`

  - Update an existing vector

- `delete(id: str) -> None`

  - Remove a vector from the database

- `exists(id: str) -> bool`

  - Check if a vector ID exists

- `search(query_vector: np.ndarray, k: int = 10, filter: dict = None) -> List[Dict]`

  - Search for k nearest neighbors
  - Filter by metadata: `filter={"category": "A", "score": {"$gt": 0.9}}`

- `search_batch(query_vectors: List[np.ndarray], k: int = 10) -> List[List[Dict]]`

  - Batch search for multiple query vectors

- `rebuild_index() -> None`

  - Rebuild the index (useful after bulk updates)

- `get_stats() -> Dict`

  - Get database statistics (size, index type, etc.)

- `save(path: str) -> None`

  - Save database to disk

- `load(path: str) -> VectorDB`
  - Load database from disk (class method)

## Similarity Metrics

### Cosine Similarity

Measures the cosine of the angle between two vectors. Range: [-1, 1]

- Best for: Normalized vectors, text embeddings
- Formula: `cos(θ) = (A · B) / (||A|| * ||B||)`

### Euclidean Distance (L2)

Straight-line distance between two points. Range: [0, ∞)

- Best for: Spatial data, when magnitude matters
- Formula: `√Σ(Ai - Bi)²`

### Dot Product

Scalar product of two vectors. Range: (-∞, ∞)

- Best for: When vectors are already normalized
- Formula: `Σ(Ai * Bi)`

### Manhattan Distance (L1)

Sum of absolute differences. Range: [0, ∞)

- Best for: Sparse vectors, when differences in all dimensions matter equally
- Formula: `Σ|Ai - Bi|`

## Examples

See the `examples/` directory for more detailed usage examples:

- `basic_usage.py`: Simple CRUD operations
- `advanced_usage.py`: Complex queries and filtering
- `benchmark.py`: Performance testing

## Performance Considerations

- **HNSW Index**: Recommended for datasets > 10,000 vectors
- **Brute Force**: Suitable for datasets < 10,000 vectors
- **Batch Operations**: Use `add_batch()` for inserting multiple vectors
- **Index Rebuilding**: Rebuild index after bulk deletions or updates

## Development

### Running Tests

```bash
pytest tests/
```

### Code Formatting

```bash
black vec_db/
```

## License

MIT License

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
