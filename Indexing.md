---
Created on: 2024-04-23
Updated on: 2024-04-23
---



## What is a vector index?
A vector index is a data structure used in computer science and information retrieval to efficiently store and retrieve high-dimensional vector data, enabling fast similarity searches and nearest neighbour queries.

```ad-question
1. How does vector index differs from vector embedding?
2. How does the vector database looks from inside? 
	1. Rows and columns of DB table
```

### Role in GenAI
Generative AI models can be tailored to specific use cases by providing them with additional context as well as long-term memory. A common pattern to provide this extra context is called _Retrieval Augmented Generation (RAG)_.
For many use cases, RAG is implemented by creating a _set of vector embeddings_ that encode semantic information about the datasets the GenAI application will use, and then searching and retrieving relevant objects from that dataset of vector embeddings to provide back to the GenAI model.
A vector index is a critical piece of the puzzle for implementing RAG in a GenAI application. 

> A ==vector index== is a data structure that enables fast and accurate search and retrieval of vector embeddings from a large dataset of objects.


## Understanding Vector indexing
- The purpose of a vector index is to search and retrieve data from a large set of vectors.
- Vector representations of data bring context to GenAI models and applications.
- A vector index enables us to find the specific data we are looking for in large sets of vector representations easily.
- Embeddings are a mathematical representation of data that captures the meaning of the object.

![Sentence similarity graph](https://cdn.sanity.io/images/bbnkhnhl/production/9831fce95d3578cc49b125c033b7c38ed9b41aed-1999x661.png?w=3840&q=75&fit=clip&auto=format)

In order to search and retrieve data from a set of embeddings, we need to define a method to compare two vectors. This is often called a _similarity measure_ or _similarity metric_. These metrics determine that two vectors are nearly the same by comparing the distance or angle between them. Common similarity metrics used in [vector search](https://www.datastax.com/guides/what-is-vector-search) include:
- **Cosine similarity:**  Measure the angle between the two vectors. Values range from -1 to 1. For a value of 1, both vectors point in the same direction, -1 they point in opposite directions, and 0 they are orthogonal (perpendicular).
- **Dot product / inner product:** Measures how well two vectors align with each other.  Values range from -∞ to ∞. Positive values indicate the vectors are in the same direction, negative values that they are in opposite directions, and 0 are orthogonal.
- **Euclidean distance:** Measures the distance between two vectors.  Values range from 0 to ∞. Zero means the vectors are identical and larger numbers are further apart.

## How does a vector index work?
In traditional databases and indexes, we store data as a row representing some fact or concept, and a set of columns that describe that concept in more detail or link us to supporting tables that contain more information. These data are scalar, meaning they have just a single value instead of vector data, which contains multiple values.

When we query the scalar index to retrieve rows or records, we generally query for exact matches. The power of indexes using vector embeddings that capture semantic information is we can instead search the index for approximate matches. We provide a vector as input and ask the vector index to return other vectors that are similar to the input or query vector. This allows us to search large datasets of vectors very quickly. The class of algorithms used to build and search vector indexes is called _Approximate Nearest Neighbor (ANN)_ search.

ANN algorithms rely on a similarity measure to determine the nearest neighbors. The vector index must be constructed based on a particular similarity metric. In order to build the vector index, we select a similarity metric and a method to create the index. 

## Understanding common indexing methods
We can explore implementations of different index strategies using the [Facebook AI Similarity Search (FAISS)](https://github.com/facebookresearch/faiss) library, a popular vector index library. Note that FAISS is only an index and not a vector database.

### 1. Flat indexing
Flat indexing is an index strategy where we simply _store each vector as is, without modifications_.

Flat indexing is simple, easy to implement, and provides perfect accuracy. The downside is it is slow. In a flat index, the similarity between the query vector and every other vector in the index is computed.

We then return the K vectors with the smallest similarity score (most closest).

Flat indexing is the right choice when perfect accuracy is required and speed is not a consideration.  If the dataset we are searching is small, flat indexing may also be a good choice as the search speed can still be reasonable.

### 2. Locally Sensitive Hashing (LSH) indexes
Locality Sensitive Hashing is an indexing strategy that optimizes for speed and finding an approximate nearest neighbor, instead of doing an exhaustive search to find the actual nearest neighbor as is done with flat indexing.

The index is built using a hashing function. Vector embeddings that are nearby each other are hashed to the same bucket. We can then store all these similar vectors in a single table or bucket.

When a query vector is provided, its nearest neighbors can be found by hashing the query vector, and then computing the similarity metric for all the vectors in the table for all other vectors that hashed to the same value. This results in a much smaller search compared (less number of vectors to compare) to flat indexing where the similarity metric is computed over the whole space, greatly increasing the speed of the query.

### 3. Inverted file (IVF) indexes
Inverted file (IVF) indexes are similar to LSH in that the goal is to first map the query vector to a smaller subset of the vector space and then only search that smaller space for approximate nearest neighbors. 

In LSH that subset of vectors was produced by a hashing function. In IVF, the vector space is first partitioned or clustered, and then centroids of each cluster are found. For a given query vector, we then find the closest centroid. Then for that centroid, we search all the vectors in the associated cluster.

Note that there is a potential problem when the query vector is near the edge of multiple clusters. In this case, the nearest vectors may be in the neighboring cluster. In these cases, we generally need to search multiple clusters.

### 4. Hierarchical Navigable Small Worlds (HNSW) indexes
Hierarchical Navigable Small World (HNSW) is one of the most popular algorithms for building a vector index. It is very fast and efficient. We won’t get into the full details of how to implement HNSW works as it is a bit complicated, but we’ll hit some of the key points here.

HNSW is a multi-layered graph approach to indexing data. At the lowest level, every vector in the index is captured.  As we move up layers in the graph, data points are grouped together based on similarity to exponentially reduce the number of data points in each layer.  In a single layer, points are connected based on their similarity.  Data points in each layer are also connected to data points in the next layer.

To search the index, we first search for the highest layer of the graph. The closest match from this graph is then taken to the next layer down where we again find the closest matches to the query vector. We continue this process until we reach the lowest layer in the graph.
--- start-multi-column: ExampleRegion1  

![Hierarchical Navigable Small World Layers](https://cdn.sanity.io/images/bbnkhnhl/production/22b43f201fae583380cb0a831add6f34b1b89561-1999x1999.png?w=3840&q=75&fit=clip&auto=format)

--- end-column ---

![Hierarchical Navigable Small World Nearest Neighbor](https://cdn.sanity.io/images/bbnkhnhl/production/b05e56108a82b042065001effbec2f1abf904346-1999x1999.png?w=3840&q=75&fit=clip&auto=format)

--- end-multi-column


## Difference between Vector Index and Vector Embeddings?
Vector indexing and vector embedding are two concepts that are often used in the context of machine learning and data science, particularly in natural language processing (NLP) and information retrieval (IR). They _serve different purposes and are used in different contexts_, but both involve representing data in a vector space.

### Vector Indexing
Vector indexing refers to the process of organising and retrieving data points (vectors) in a way that allows for efficient similarity searches. This is particularly useful in applications where you need to find the most similar items to a given query item. The goal of vector indexing is to quickly identify which vectors in a dataset are most similar to a given query vector.
- **Purpose**: Efficiently finding the most similar vectors to a given query vector.
- **Techniques**: Various indexing methods are used, such as k-d trees, ball trees, and approximate nearest neighbor (ANN) algorithms like Locality-Sensitive Hashing (LSH) and Product Quantization (PQ).
- **Use Cases**: Information retrieval, recommendation systems, and similarity search in large datasets.

### Vector Embedding
Vector embedding, on the other hand, is the process of representing discrete data, such as words, phrases, or entities, in a continuous vector space. The goal is to capture the semantic or syntactic relationships between the data points in a way that allows for meaningful comparisons and operations in the vector space.
- **Purpose**: Representing discrete data in a continuous vector space to capture relationships.
- **Techniques**: Various embedding techniques are used, such as Word2Vec, GloVe, and FastText for words, and entity embeddings for entities. These techniques learn embeddings from data, capturing the context in which words or entities appear.
- **Use Cases**: Natural language processing (NLP), information retrieval (IR), and recommendation systems.

### Key Differences
- **Objective**: Vector indexing is about organizing and retrieving vectors efficiently, while vector embedding is about representing discrete data in a continuous vector space.
- **Data**: Vector indexing deals with any type of vectors, while vector embedding specifically focuses on representing discrete data like words or entities.
- **Techniques**: Vector indexing uses algorithms for efficient search, while vector embedding uses techniques that learn representations from data.
- **Use Cases**: Vector indexing is used in applications that require fast similarity searches, while vector embedding is used in applications that require understanding the semantic or syntactic relationships between data points.

In summary, while both vector indexing and vector embedding involve working with vectors, they serve different purposes and are used in different contexts. Vector indexing is about efficiently finding similar vectors, while vector embedding is about representing discrete data in a way that captures their relationships.




# Source links
1. [What is a Vector Index?](https://www.datastax.com/guides/what-is-a-vector-index)