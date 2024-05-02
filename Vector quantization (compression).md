---
Created on: 2024-04-23
Updated on: 2024-04-23
---
```ad-note
1. Define vector quantization in detail.
```


**Vector quantization** reduces the memory footprint of the [vector index](https://weaviate.io/developers/weaviate/concepts/vector-index) by compressing the vector embeddings, and thus reduces deployment costs and improves the speed of the vector similarity search process.


## What is Quantization?
In general, quantization techniques reduce the memory footprint by representing numbers with lower precision numbers, like rounding a number to the nearest integer. 
In neural networks, quantization reduces the values of the weights or activations of the model stored as a 32-bit floating-point number (4 bytes) to a lower precision number, such as an 8-bit integer (1 byte).


### Product Quantization (PQ)
[Product quantization](https://ieeexplore.ieee.org/document/5432202) is a multi-step quantization technique.

PQ reduces the size of each vector embedding in two steps. First, it reduces the number of vector dimensions to a smaller number of "segments", and then each segment is quantized to a smaller number of bits from the original number of bits (typically a 32-bit float).

PQ makes tradeoffs between recall, performance, and memory usage. This means a PQ configuration that reduces memory may also reduce recall. There are similar trade-offs when you use HNSW without PQ. If you use PQ compression, you should also tune HNSW so that they compliment each other.

In PQ, the original vector embedding is represented as a product of smaller vectors that are called 'segments' or 'subspaces.' Then, each segment is quantized independently to create a compressed vector representation.

![PQ illustrated](https://weaviate.io/assets/images/pq-illustrated-fb5fc3da31c00575947879a552861ae7.png "PQ illustrated")

After the segments are created, there is a training step to calculate `centroids` for each segment. By default, Weaviate clusters each segment into 256 centroids. The centroids make up a codebook that Weaviate uses in later steps to compress the vector embeddings.

Once the codebook is ready, Weaviate uses the id of the closest centroid to compress each vector segment. The new vector representation reduces memory consumption significantly. Imagine a collection where each vector embedding has 768 four byte elements. Before PQ compression, each vector embedding requires `768 x 4 = 3072` bytes of storage. After PQ compression, each vector requires `128 x 1 = 128` bytes of storage. The original representation is almost 24 times as large as the PQ compressed version. (It is not exactly 24x because there is a small amount of overhead for the codebook.)

#### Segments
The PQ `segments` controls the tradeoff between memory and recall. A larger `segments` parameter means higher memory usage and recall. An important thing to note is that the segments must divide evenly the original vector dimension.

Below is a list segment values for common vectorizer modules:

| Module      | Model                                   | Dimensions | Segments               |
| ----------- | --------------------------------------- | ---------- | ---------------------- |
| openai      | text-embedding-ada-002                  | 1536       | 512, 384, 256, 192, 96 |
| cohere      | multilingual-22-12                      | 768        | 384, 256, 192, 96      |
| huggingface | sentence-transformers/all-MiniLM-L12-v2 | 384        | 192, 128, 96           |

### Binary Quantization (BQ)
**Binary quantization** is a quantization technique that converts each vector embedding to a binary representation. The binary representation is much smaller than the original vector embedding. Usually each vector dimension requires 4 bytes, but the binary representation only requires 1 bit, representing a 32x reduction in storage requirements. This works to speed up vector search by reducing the amount of data that needs to be read from disk, and simplifying the distance calculation.

The tradeoff is that BQ is lossy. The binary representation by nature omits a significant amount of information, and as a result the distance calculation is not as accurate as the original vector embedding.

Some vectorizers work better with BQ than others. Anecdotally, we have seen encouraging recall with Cohere's V3 models (e.g. `embed-multilingual-v3.0` or `embed-english-v3.0`), and OpenAI's `ada-002` model with BQ enabled. We advise you to test BQ with your own data and preferred vectorizer to determine if it is suitable for your use case.

Note that when BQ is enabled, a vector cache can be used to improve query performance. The vector cache is used to speed up queries by reducing the number of disk reads for the quantized vector embeddings. Note that it must be balanced with memory usage considerations, with each vector taking up `n_dimensions` bits.

#### Over-fetching / re-scoring
When using BQ, Weaviate will conditionally over-fetch and then re-score the results. This is because the distance calculation is not as accurate as the original vector embedding.

This is done by fetching the higher of the specified query limit, or the rescore limit objects, and then re-score them using the full vector embedding. As a concrete example, if a query is made with a limit of 10, and a rescore limit of 200, Weaviate will fetch `max(10, 200) = 200` objects, and then re-score the top 10 objects using the full vector. This works to offset some of the loss in search quality (recall) caused by compression.


### Rescoring
Quantization inherently involves some loss information due to the reduction in information precision. To mitigate this, Weaviate uses a technique called rescoring, using the uncompressed vectors that are also stored alongside compressed vectors. Rescoring recalculates the distance between the original vectors of the returned candidates from the initial search. This ensures that the most accurate results are returned to the user.

In some cases, rescoring also includes over-fetching, whereby additional candidates are fetched to ensure that the top candidates are not omitted in the initial search.









# Legends
| Short hand | Full form                           |
| ---------- | ----------------------------------- |
| PQ         | Product Quantization                |
| BQ         | Binary Quantization                 |
| HNSW       | Hierarchical Navigable Small Worlds |

# Source links
1. [Weaviate: Vector Quantization (compression)](https://weaviate.io/developers/weaviate/concepts/vector-quantization#binary-quantization)


