# Hybrid Search with SingleStore — Amazon Reviews Demo

This repository demonstrates how to run **real-time hybrid search** (full-text + vector similarity + combined scoring) on a dataset of Amazon product reviews using [SingleStore Helios](https://www.singlestore.com/helios/?utm_source=Aasawari&utm_medium=blog&utm_campaign=elastic&utm_content=singlestoresearch).

The notebook [`SingleStore10Kreviews.ipynb`](./SingleStore10kReviewsipynb) shows how to:

- Connect to a SingleStore Helios workspace
- Load a dataset of Amazon reviews
- Generate embeddings with `sentence-transformers`
- Store embeddings directly in a SingleStore `VECTOR` column
- Perform:
  - **Full-text search** using `MATCH ... AGAINST`
  - **Vector similarity search** using `DOT_PRODUCT` and cosine similarity
  - **Hybrid search** that blends text and semantic scores in a single SQL query

---

## 🔍 Why SingleStore for Hybrid Search?

Traditional search engines like Elasticsearch handle keyword/BM25 search well, but become complex when you add vectors, SQL-style filters, and analytics.  
SingleStore solves this by combining:

- **Native vector search** with `DOT_PRODUCT` and cosine similarity
- **Full-text search** directly in SQL
- **Hybrid scoring** with tunable weights or normalization
- **Real-time visibility** — no refresh cycles needed
- **One system** for search + analytics + transactions

---

## 📊 Search Comparison: SingleStore vs. Elasticsearch

Below are the results from running the same queries (`"dog food quality"`) on **100 Amazon reviews**.  
Each query type was executed on both SingleStore and Elasticsearch.

| Search Type | Elasticsearch | SingleStore |
|-------------|---------------|-------------|
| **Full-Text Search** | Execution time: **1.67s** <br> Strong BM25 ranking, but noisy matches at scale | Execution time: **0.38s** <br> SQL `MATCH ... AGAINST`, precise results |
| **Vector Search** | Execution time: **1.82s** <br> Needs `dense_vector` mapping + `knn` API | Execution time: **0.37s** <br> Native vector column, simple query |
| **Hybrid Search** | Execution time: **0.32s** <br> Requires `function_score` + `script_score`, opaque scoring | Execution time: **0.35–0.65s** <br> Simple weighted SQL expression, transparent & tunable |
| **Conclusion** | Great for text search, but complex and slower for hybrid/vector workloads | Unified, real-time hybrid search with SQL. Faster, simpler, and consistent |

---

## ▶️ Example Queries in SingleStore

### 🔹 Full-Text Search
```sql
SELECT id, Summary, Text, 
       MATCH(Text) AGAINST ('dog food quality') AS relevance
FROM amazon_reviews
ORDER BY relevance DESC
LIMIT 5;
````

### 🔹 Vector Search

```sql
SELECT id, Summary, Text, 
       DOT_PRODUCT(embedding_vector, JSON_ARRAY_PACK('[0.1, 0.2, ...]')) AS similarity
FROM amazon_reviews
ORDER BY similarity DESC
LIMIT 5;
```

### 🔹 Hybrid Search

```sql
SELECT id, Summary, Text,
       0.5 * MATCH(Text) AGAINST ('dog food quality') +
       0.5 * DOT_PRODUCT(embedding_vector, JSON_ARRAY_PACK('[0.1, 0.2, ...]')) AS hybrid_score
FROM amazon_reviews
ORDER BY hybrid_score DESC
LIMIT 5;
```

---

## ▶️ How to Run the Notebook

### Prerequisites

* Python 3.9+
* Install dependencies:

  ```bash
  pip install singlestoredb sentence-transformers numpy
  ```
* A SingleStore Helios workspace (free tier available)

### Steps

1. Open `singlestore-100-amazon.ipynb`
2. Update the connection string with your Helios credentials
3. Run the cells to:

   * Load the dataset
   * Generate embeddings
   * Execute full-text, vector, and hybrid search queries
4. Observe results and execution times

---

## ✅ Key Takeaway

With SingleStore, you don’t need multiple systems for text, vector, and analytics. Everything runs **in one SQL engine** with **real-time freshness**, **lower latency**, and **simpler developer experience**.

---

## 📌 CTA

Finally, learn more about SingleStore — visit our [official documentation](https://docs.singlestore.com/?utm_source=Aasawari&utm_medium=blog&utm_campaign=elastic&utm_content=singlestoresearch).
You can also [register for a SingleStore webinar](https://www.singlestore.com/resources/webinars/?utm_source=Aasawari&utm_medium=blog&utm_campaign=elastic&utm_content=singlestoresearch) and learn about the latest concepts and technologies with hands-on experience.
