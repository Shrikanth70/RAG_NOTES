# Note 11: Cosine Similarity & Distance Metrics

## 1. Concept Overview
To perform semantic search, the vector database must mathematically measure the "closeness" between the query vector and the document vectors. 

The three primary distance metrics used are:
1. **Cosine Similarity**: Measures the cosine of the angle between two vectors in high-dimensional space. It evaluates directional alignment rather than length.
2. **Euclidean Distance ($L_2$ Distance)**: Measures the straight-line distance between two coordinate points.
3. **Dot Product (Inner Product)**: Multiplies matching coordinates and sums them. If vectors are normalized to unit length, Dot Product is identical to Cosine Similarity.

---

## 2. Why It Exists
Computers cannot check if two paragraphs "mean" the same thing by reading them word-by-word. Instead, we represent them as arrow coordinates pointing from a central origin. 

If two paragraphs talk about the same topic, their arrows will point in the same direction. Cosine Similarity calculates the angle between these arrows. An angle of $0^\circ$ (pointing in the exact same direction) yields a cosine similarity score of $1.0$. An angle of $90^\circ$ (orthogonal, totally unrelated) yields a score of $0.0$. An angle of $180^\circ$ (opposite meanings) yields a score of $-1.0$.

---

## 3. Real-World Analogy
Imagine two people standing in a field pointing at stars in the night sky:
- **Cosine Similarity**: Checks if they are pointing at the *same star* (same direction/angle), regardless of how long their arms are.
- **Euclidean Distance**: Measures the physical distance between their index fingers. If one person has a much longer arm, the finger distance is large even if they are pointing at the exact same star.
- **Dot Product**: Combines both their directions and their arm lengths.

---

## 4. Technical Explanation
The formula for Cosine Similarity between vector $A$ and vector $B$ is:

$$\text{Similarity}(A, B) = \cos(\theta) = \frac{A \cdot B}{\|A\| \|B\|} = \frac{\sum_{i=1}^{n} A_i B_i}{\sqrt{\sum_{i=1}^{n} A_i^2} \sqrt{\sum_{i=1}^{n} B_i^2}}$$

In Python, this is computed as:

```python
import numpy as np

def cosine_similarity(a: list[float], b: list[float]) -> float:
    vec_a = np.array(a)
    vec_b = np.array(b)
    dot_product = np.dot(vec_a, vec_b)
    norm_a = np.linalg.norm(vec_a)
    norm_b = np.linalg.norm(vec_b)
    
    if norm_a == 0.0 or norm_b == 0.0:
        return 0.0
    return float(dot_product / (norm_a * norm_b))
```

If we normalize our vectors during ingestion so that $\|A\| = 1.0$ and $\|B\| = 1.0$, the denominator becomes $1.0$, and the formula simplifies to a basic dot product: $\sum A_i B_i$. This is highly optimized in modern CPU instruction sets.

---

## 5. Industry Best Practices
- **Use Cosine Similarity for Text**: For text embeddings, Cosine Similarity is the industry standard because it is invariant to document length. A short sentence and a long paragraph on the same topic will have highly aligned directions even if their vector lengths differ.
- **Normalize Vectors in Advance**: Normalize vectors to unit length during ingestion. That way, during search, the database can use the Dot Product index, which runs much faster than computing full Cosine angles on the fly.
- **Convert Distance to Score**: Some databases return Cosine *Distance* ($1 - \text{Similarity}$). Ensure you convert it back to Similarity ($1 - \text{Distance}$) if displaying confidence percentages to users.

---

## 6. Common Mistakes
- **Mixing Metrics**: Setting up your database index with Euclidean Distance but querying it assuming Cosine Similarity scores. This yields garbled rankings.
- **Ignoring Normalization in Dot Product**: Querying a dot-product index with unnormalized vectors. A very long, wordy document might yield a massive vector length and score highly on every query just because of its size, rather than its relevance.

---

## 7. Visual Diagram (ASCII)
```
          y ^
            |      / Vector A (Short document on Cars)
            |    / 
            |  / _  Angle theta (small = highly similar)
            |/     \ 
            +---------> Vector B (Long document on Cars)
            | \
            |   \
            |     \ Vector C (Unrelated document on Cooking)
            +---------------------------------------------> x
```

---

## 8. Interview Questions
1. **Why is Cosine Similarity preferred over Euclidean Distance for text embeddings?**
   *Answer*: Text documents vary in length. Euclidean distance is sensitive to vector magnitude (length), meaning a long document that repeats a topic many times will have a large coordinate distance from a short document on the same topic. Cosine similarity only measures the angle (direction) of the vectors, making it length-invariant and far more accurate for comparing texts of different lengths.
2. **What is the mathematical relationship between Dot Product and Cosine Similarity?**
   *Answer*: Cosine similarity is the dot product of two vectors divided by the product of their magnitudes (lengths). If both vectors are normalized to unit length (magnitude of 1), the cosine similarity is exactly equal to the dot product.

---

## 9. Key Takeaways
- Cosine Similarity measures the angle between vectors, ignoring length.
- Scores range from -1.0 (opposite) to 1.0 (identical), but for text, they usually fall between 0.0 and 1.0.
- Normalizing vectors turns Cosine Similarity into Dot Product, which is computationally faster.
- Euclidean Distance measures straight-line distance; it is sensitive to document length.

---

## 10. Beginner Summary
Cosine similarity is like a compass. It doesn't care how far you walk (document length); it only cares which direction you are facing. If two documents are facing the same direction (talking about the same topic), their compass needles align, and the system knows they are related.
