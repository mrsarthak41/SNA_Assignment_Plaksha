# Large Scale Social Network Analysis: Wikipedia Vote Network

A complete, end-to-end Python pipeline using `pandas` and `NetworkX` to analyze user influence and community structure in real-world social networks.

---

## 1. Problem Formulation & Objectives

Social networks often exhibit power-law degree distributions where a small minority of influential users drive platform decisions and interactions. The objective of this project is to build an automated, end-to-end analytical pipeline that:
1. Extract and preprocess real-world directed network data.
2. Construct and mathematically justify the graph topology.
3. Evaluate and compare distinct centrality metrics (In-Degree, Betweenness, Eigenvector, and PageRank).
4. Detect community structures and verify influencer stability across groups.
5. Analyze theoretical scalability under 100× graph size expansion.

---

## 2. Dataset Proposal & Preprocessing

### Dataset Selection
* **Selected Dataset:** Wikipedia Vote Network (Stanford SNAP Repository)
* **Expected Graph Size:** ~7,115 Nodes | ~103,689 Edges

### Why Wikipedia Vote Network over Alternatives?
When evaluating potential datasets, two alternative SNAP datasets were considered:
* **GitHub Social Network:** ~37,700 nodes
* **Reddit Hyperlink Network:** ~55,000 nodes

**Justification for Choice:**
1. **Semantic Clarity:** The Wikipedia Vote network represents explicit, directed peer-voting events for administrator elections. Directed edges ($A \to B$) have a unambiguous interpretation of trust/authority, unlike the dense comment-reply threads of Reddit.
2. **Computational Tractability:** Exact calculation of global graph metrics—specifically Betweenness Centrality with time complexity $O(V \cdot E)$—becomes computationally prohibitive on single-node setups when $V > 30,000$. The Wikipedia dataset offers a scale large enough to display non-trivial social dynamics without requiring distributed clusters (e.g., Apache Spark).

### Expected Challenges
* **High Computational Burden:** Exact Betweenness Centrality scale quadratically with node/edge size.
* **Graph Topology Issues:** Disconnected islands and "sink" nodes (users who receive votes but cast none) can distort traditional random-walk metrics like PageRank without proper damping.

### Preprocessing Strategy
1. **Automated Download:** Extracted directly via Python's `urllib`.
2. **Comment Filtering:** Comments starting with `#` are ignored using `pandas.read_csv`.
3. **Loop Removal:** Self-loops ($v \to v$) are stripped to prevent self-reinforcing centrality scores.
4. **LWCC Extraction:** Isolated sub-graphs are removed by isolating the **Largest Weakly Connected Component (LWCC)** to ensure metric consistency.

---

## 3. Methodology & Implementation

### Graph Construction & Justification
* **Directed:** A vote from User A to User B is asymmetric ($A \to B \neq B \to A$).
* **Unweighted:** Every vote represents a uniform binary action (1 vote = 1 edge weight).

### Centrality Metrics Comparison & $k$-Sampling Experiment
Four key centrality measures were computed to evaluate top influencers:

| Metric | Focus / Interpretation | Computational Complexity |
| :--- | :--- | :--- |
| **In-Degree** | Direct popularity (total votes received) | $O(V + E)$ |
| **Betweenness** | Information bottleneck / bridge role | $O(V \cdot E)$ exact vs. $O(k \cdot E)$ sampled |
| **Eigenvector** | Connectedness to other highly connected nodes | $O(E)$ per iteration |
| **PageRank** | Global authority with random-jump damping ($\alpha=0.85$) | $O(V + E)$ per iteration |

#### Key Empirical Finding: Betweenness Centrality Efficiency
During implementation, an experiment was conducted on the trade-off between exact vs. stochastic sampling for Betweenness Centrality:
* **Without $k$ (Exact):** Running full evaluation required **~230 seconds** due to exhaustive shortest-path enumeration across all $\approx 7,000$ nodes ($O(V \cdot E)$).
* **With $k=100$:** Execution dropped to **under 2 seconds** while preserving the rank order of top global bridges.
* **With $k=1000$:** Improved estimation precision with a modest runtime (~15 seconds).

*Conclusion:* Stochastic sampling ($k=100$) transforms quadratic computational bottlenecks into linear $O(k \cdot E)$ tasks, essential for scaling.

---

## 4. Community Detection & Influencer Stability

* **Algorithm:** Louvain Method (Optimizing graph modularity).
* **Graph Conversion:** Graph converted to **Undirected** view (required mathematically by Louvain).
* **Key Finding:** Top PageRank influencers do not concentrate within a single modular community. Instead, they are distributed across separate, distinct communities, demonstrating that top-tier PageRank requires global cross-community reach rather than localized density.

---

## 5. Scalability & Complexity Analysis (100× Expansion)

If the dataset scales by **100×** (e.g., $V' = 100V, E' = 100E$):

1. **PageRank $O(V + E)$:** Scales **linearly**. Runtime increases by $\approx 100\times$, remaining computationally feasible on a single node.
2. **Exact Betweenness $O(V \cdot E)$:** Scales **quadratically**. $(100V) \cdot (100E) = 10,000 \times$ increase in operations. The ~230s execution would expand to **>26 days**, leading to out-of-memory or kernel crash errors.
3. **Mitigation:** Employing stochastic sampling ($k=100$) or distributed graph engines (GraphX / Networkit) is mandatory at 100× scale.

---

## 6. Project Structure & Reproducibility

```text
├── wiki-Vote.txt.gz          # Downloaded network dataset
├── analysis_pipeline.ipynb   # Main execution notebook
├── README.md                 # Project documentation
```

### Dependencies
```bash
pip install pandas networkx
```

### How to Run
1. Open `analysis_pipeline.ipynb` in Jupyter Lab.
2. Run all cells sequentially. The dataset will auto-download and execute end-to-end.

---

## 7. AI Disclosure & Acknowledgments

* **AI Assistance:** Gemini (Google AI) was utilized as an interactive tutor and pair-programming assistant during development for:
  * Code learning, refining and optimization.
  * Conceptual learning of graph theory algorithms and Big-O computational bounds.
  * Text formatting and markdown structure refinement.
* All code logic was implemented, executed, and validated independently.
