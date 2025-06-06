# GSoC’25 Proposal NetworkX - Adding embarrassingly parallel graph algorithms in nx-parallel

## Abstract

This proposal aims to enhance the nx-parallel backend of NetworkX by implementing a suite of embarrassingly parallel algorithms over the 12-week GSoC coding period. These algorithms exhibit minimal inter-process communication and can be independently executed across graph nodes or edges, making them ideally suited for parallel execution. Additionally, throughout the project, I will be actively reviewing and revisiting existing parallel implementations in the nx-parallel repository to identify opportunities for optimization and resolving bugs.

## Deliverables

The following algorithms have been identified as embarrassingly parallel and are scheduled for implementation

- **Updating timing script**

  Generate consistent heatmaps to visualize performance, and explore shared memory strategies to improve efficiency during parallel execution. (ref. [PR#114](https://github.com/networkx/nx-parallel/pull/114)). 

- [`number_attracting_components`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.components.number_attracting_components.html), [`number_connected_components`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.components.number_connected_components.html), [`number_strongly_connected_components`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.components.number_strongly_connected_components.html), [`number_weakly_connected_components`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.components.number_weakly_connected_components.html)

  The parallelisation approach for these functions mirrors that of `number_of_isolates`. Rather than parallelising the internal logic of itself — which remains sequential — we parallelise the counting step after obtaining the list of attracting components. Since each component is independent, the counting can be parallelised but the processing time of each chunk would be small (only the sum would be computed for each chunk), so chunking would not yield any speedups. Adding a `should_run` parameter here would be beneficial for this reason.

- [`triangles`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.cluster.triangles.html)
  
  This function computes the number of triangles each node participates in. The existing sequential implementation in NetworkX is already optimized using the late neighbors strategy — a node only considers neighbors that appear later in a consistent order. This ensures each triangle is counted exactly once and avoids redundant comparisons. Chunking is preferred over assigning one node per job, as spawning a large number of parallel tasks for each node can introduce significant overhead killing the speedup. Observed speedups are in the range of 1.5 to 2 times the sequential implementation on my 8-core machine. </br>
  *Currently in progress* (ref. [PR#106](https://github.com/networkx/nx-parallel/pull/106))

- [`clustering`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.cluster.clustering.html#networkx.algorithms.cluster.clustering)

  Computes the clustering coefficient for each node, measuring the tendency of a node’s neighbors to form triangles. The main parallelization strategy chunks the list of nodes and processes each chunk independently (Default chunking). Each worker computes clustering values for its assigned nodes restricted to the chunk. If node degrees are skewed, we might have to tweak the chunking logic. I would have to experiment with this during the implementation to reach a particular conclusion. A potential issue that arises here is that multiple copies of the graph are created during its execution across the cores which would lead to memory inefficiency in large graphs. This would need to be handled. For very small graphs, parallelism overhead may outweigh benefits. 

- [`average_clustering`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.cluster.average_clustering.html)

  Averages the clustering coefficients across all nodes. It utilises the parallel implementation of `clustering`.After obtaining the clustering coefficients, the final step reduces these values by computing their average using a sum and len. To better understand the trade-offs, I plan to experiment with two versions:
    </br> Both clustering and the sum/len aggregation are done in parallel.
    </br> Only clustering is parallelized, while aggregation remains sequential. </br>
  To control whether parallel backend should be used for this function at all, adding the [`should_run`](https://github.com/networkx/nx-parallel/issues/77) parameter here could be helpful.

- `_apply_prediction`

  This internal utility function is the core engine behind most link prediction algorithms in NetworkX. Instead of applying the function sequentially over all edge pairs, the list of node pairs can be chunked, and the prediction function can be executed in parallel on each chunk. Instead of applying parallelism to individual functions like `jaccard_coefficient`, `adamic_adar_index`, `preferential_attachment`etc. Upon implementing the algorithm, I'd like to try and see if reducing the size of default chunking would yield better results since each of the functions that do the work are light-weight.

- [`harmonic_centrality`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.centrality.harmonic_centrality.html)

  Measures the closeness of a node to all other reachable nodes based on the inverse of the shortest-path distances. It would involve computing shortest-path distances from each `source` and updating centrality scores for the relevant targets (`nbunch`). Since each source's computation is independent, the problem is embarrassingly parallel across sources. A default chunking strategy can be applied by dividing the sources into n_jobs chunks and processing them in parallel. After computing partial centrality scores in each job, the results can be aggregated (e.g., using `Counter.update` or dictionary merging) into the final output dictionary.

- [`average_neighbor_degree`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.assortativity.average_neighbor_degree.html) 

  Computes the mean degree of neighbors for each node (source, target). Since each node’s computation is independent, the function is embarrassingly parallel. The list of nodes is chunked and processed in parallel, where each worker computes average neighbor degrees for its chunk. While computation per node is light, spawning one task per node is inefficient — chunking mitigates that. However, since the full graph is passed to each worker, this may result in multiple copies being pickled, leading to memory inefficiency on large graphs. To reduce overhead, I will have to look into incorporating shared-memory strategies on graphs where it would just access the graph from a shared memory (since it is read-only). Adding a `should_run` flag may help manage this trade-off. 

## Optional Extensions (If Time Permits)

- Add these embarassingly parallel algorithms after thorough research.
  - [`load_centrality`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.centrality.load_centrality.html)
  - [`eccentricity`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.distance_measures.eccentricity.html)
- After discussing the scope of adding non-embarassingly parallel algorithms, work on 
  **Delta-Stepping** : A parallel single-source shortest paths (SSSP) algorithm, a variant of Dijkstra's algorithm, maintaining buckets representing priority ranges of size Δ. </br>
  *See:* https://www.cs.utexas.edu/~pingali/CS395T/2013fa/papers/delta-stepping.pdf

- Adding user guide tutorials for using nx-parallel. 

## Project Timeline

 **Weeks 1–4**  
 - Work on the timing script<br> 
 - Parallel Implementations of `triangles`,  `number_attracting_components`,`number_connected_components`, `number_strongly_connected_components`, `number_weakly_connected_components`.

 **Weeks 5-8**
 - Parallel Implementations of `clustering`, `average_clustering` and `_apply_prediction` 

 **Weeks 9–12** 
 - Parallel implementations of `harmonic_centrality`, `average_neighbor_degree`.
 
 Throughout the project, I will also contribute to independent PRs and use review periods to plan and research the next implementation steps.

## Why NetworkX?

I’ve always been drawn to logical thinking, which naturally led me to explore programming even before college. While exploring previous GSoC organizations, I noticed that many projects either did not align with my interests or were difficult to
grasp at first glance.

When I discovered NetworkX, I immediately identified it as a project of interest. It focused on graph theory and algorithms, areas in which I have a genuine interest and knowledge. I was also impressed by how clear and approachable the work with NetworkX was. After contributing, I realised that the community was very open and supportive, and this made the experience even more rewarding.

I am eager to continue working in this organization because it aligns perfectly with my interests, and I appreciate the welcoming nature of its community. Contributing to this project represents not only a good learning experience but also an opportunity to work on something that fascinates me.

## Time Availability

The Google Summer of Code timeline is mostly in sync with my university’s summer break and thus will allow me ample time to work on my project. Even after the break ends, there would be no tests or examinations, and the classes would be asynchronous, leaving me enough time to work. My awake hours are between 10:00 AM IST (4:30 AM UTC) and 11:00 PM IST (5:30 PM UTC), and I would be reachable any time between them. If I take a day off due to unexpected situations, I will inform the mentors in advance and will make up for that time in the subsequent days of the internship.

I will have my university classes from August, and their timings are 10:00 AM IST (4:30 AM UTC) to 04:00 PM IST (10:30 AM UTC). During this month, I will be available to work on the project at any time except the time I have my classes. But I would
still be reachable during that time.

I have an internship interview with D. E. Shaw for the Summer 2026 scheduled around the first week of July 2025. While the exact timeline has not yet been released, I would have to travel and may be preoccupied for 2–3 days during that week. I will inform the mentors of the exact dates as soon as my schedule is confirmed.

## Community Bonding Period contributions

- [PR#104](https://github.com/networkx/nx-parallel/pull/104)
- [PR#112](https://github.com/networkx/nx-parallel/pull/112)
- [issue#111](https://github.com/networkx/nx-parallel/issues/111)
