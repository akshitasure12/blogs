# GSoC’25 Proposal NetworkX - Adding embarrassingly parallel graph algorithms in nx-parallel

## Abstract

This proposal aims to enhance the nx-parallel backend of NetworkX by implementing a suite of embarrassingly parallel algorithms over the 12-week GSoC coding period. These algorithms exhibit minimal inter-process communication and can be independently executed across graph nodes or edges, making them ideally suited for parallel execution. Additionally, throughout the project, I will be actively reviewing and revisiting existing parallel implementations in the nx-parallel repository to identify opportunities for optimization and resolving bugs.

## Deliverables

The following algorithms have been identified as embarrassingly parallel and are scheduled for implementation :

- [`triangles`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.cluster.triangles.html)
  
  Efficiently counts the number of triangles a node is part of. Parallelism is achievable as each node's triangle count can be computed independently. </br>
  *Currently in progress* (ref. [PR#106](https://github.com/networkx/nx-parallel/pull/106))

- [`harmonic_centrality`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.centrality.harmonic_centrality.html)

  Measures the sum of the reciprocals of the shortest path distances from all other nodes to u. Computation of shortest paths from each of the source nodes is entirely independent of the others

- [`closeness_centrality`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.centrality.closeness_centrality.html)

  Measures how close a node is to all others. While requiring shortest paths, calculations for each node can be performed in parallel.

- [`clustering`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.cluster.clustering.html#networkx.algorithms.cluster.clustering)

  Computes the tendency of a node's neighbors to form triangles. It iterates over all nodes, computing triangle counts and degree values. Each of these computations for each node are fully independent of one another making it embarassingly parallel.

- [`average_clustering`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.cluster.average_clustering.html)

  Averages the clustering coefficients across all nodes. Can be derived from parallel node-level computations.

- [`jaccard_coefficient`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.link_prediction.jaccard_coefficient.html)

  Measures node similarity as the ratio of common to total neighbors. This coefficient is computed for given node pairs independently, making it embarrassingly parallel due to the lack of inter-pair dependencies.

- [`average_neighbor_degree`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.link_prediction.jaccard_coefficient.html) 

  Computes the mean degree of neighbors for each node. Since each node's average is computed independently based on its neighborhood,
  the function is highly parallelizable. 

## Optional Extensions (If Time Permits)

- [`transitive_closure_dag`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.dag.transitive_closure_dag.html)

  Computes reachability in directed acyclic graphs. The reachability from each node can be computed independently, so the operation is embarrassingly parallel.

- [`eccentricity`](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.distance_measures.eccentricity.html)

  Measures the maximum shortest path length from a node to all others. Each node's eccentricity can be computed in isolation.

- **Delta-Stepping**

  A parallel single-source shortest paths (SSSP) algorithm, a variant of Dijkstra's algorithm, maintaining buckets representing priority ranges of size Δ. </br>
  *See:* https://www.cs.utexas.edu/~pingali/CS395T/2013fa/papers/delta-stepping.pdf

- **Updating timing script**

  Generate consistent heatmaps to visualize performance, consider alternatives like `time.perf_counter()` for high-resolution wall-clock timing (ref. [issue#51](https://github.com/networkx/nx-parallel/issues/51)).

## Project Timeline

| **Period**                              | **Tasks** |
|-----------------------------------------|-----------|
| **Week 1** (June 2nd – June 8th)        | - Implement the first algorithm. <br> - Conduct initial tests and performance benchmarks. <br> - Get feedback from mentors. |
| **Week 2** (June 9th – June 15th)       | - Add the suggestions recommended. <br> - Finish working on the first algorithm. <br> - Research and implement the second algorithm. |
| **Week 3** (June 16th – June 22nd)      | - Conduct initial tests and performance benchmarks. <br> - Get feedback from mentors. |
| **Week 4** (June 23rd – June 29th)      | - Add the respective recommendations. <br> - Finish working on the second algorithm. |
| **Week 5** (June 30th – July 6th)       | - Research and implement the third algorithm. <br> - Conduct initial tests and performance benchmarks. <br> - Get feedback and continue working on the third algorithm. |
| **Week 6** (July 7th – July 13th)       | - Finish working on the third algorithm. <br> - Buffer week for improvements and debugging. |
| **Week 7** (Mid-Term Evaluation, July 14th – July 20th) | - Submit progress for mid-term evaluation. <br> - Research and implement the fourth algorithm. |
| **Week 8** (July 21st – July 27th)      | - Conduct initial tests and performance benchmarks. <br> - Get feedback and finish working on the fourth algorithm. |
| **Week 9** (July 28th – August 3rd)     | - Research and implement the fifth algorithm. <br> - Conduct initial tests and performance benchmarks. |
| **Week 10** (August 4th – August 10th)  | - Get feedback and finish working on the fifth algorithm. <br> - Research and implement the sixth algorithm. <br> - Conduct initial tests and performance benchmarks. |
| **Week 11** (August 11th – August 17th) | - Get feedback and finish working on the sixth algorithm. <br> - Buffer week for improvements and suggestions. |
| **Week 12** (August 18th – August 24th) | - Research and implement the final algorithm. <br> - Conduct initial tests and performance benchmarks. <br> - Get a review and finish working on the seventh algorithm. |
| **Final Evaluation** (August 25th – August 31st) | - Conduct final tests to verify the correctness and efficiency of all the algorithms, incorporating any final suggestions and recommendations. <br> - Create a final project report and submit it with all the work done for final review. |

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
