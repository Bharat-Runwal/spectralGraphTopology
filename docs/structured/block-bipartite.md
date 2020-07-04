---
layout: default
title: Block bipartite
parent: Structured graphs
nav_order: 1
---

# Block bipartite
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

``` r
library(spectralGraphTopology)
library(igraph)
library(viridis)
library(corrplot)
set.seed(42)

# design sythetic graph
w <- c(1, 0, 0, 1, 0, 1) * runif(6)
Laplacian <- block_diag(L(w), L(w))
Atrue <- diag(diag(Laplacian)) - Laplacian
# create graph from Adjacency matrix
bipartite <- graph_from_adjacency_matrix(Atrue, mode = "undirected", weighted = TRUE)
n <- ncol(Laplacian)
# sample data from GMRF
Y <- MASS::mvrnorm(40 * n, rep(0, n), MASS::ginv(Laplacian))
# learn graph on the basis of sampled data
graph <- learn_bipartite_k_component_graph(cov(Y), k = 2, beta = 1e2, nu = 1e2, verbose = FALSE)
graph$adjacency[graph$adjacency < 1e-2] <- 0
# Plot Adjacency matrices: true and estimated
corrplot(Atrue / max(Atrue), is.corr = FALSE, method = "square", addgrid.col = NA, tl.pos = "n", cl.cex = 1.25)
corrplot(graph$adjacency / max(graph$adjacency), is.corr = FALSE, method = "square", addgrid.col = NA, tl.pos = "n", cl.cex = 1.25)
# Build networks
estimated_bipartite <- graph_from_adjacency_matrix(graph$adjacency, mode = "undirected", weighted = TRUE)
V(bipartite)$type <- rep(c(TRUE, FALSE), 4)
V(estimated_bipartite)$type <- rep(c(TRUE, FALSE), 4)
la = layout_as_bipartite(estimated_bipartite)
colors <- viridis(20, begin = 0, end = 1, direction = -1)
c_scale <- colorRamp(colors)
E(estimated_bipartite)$color = apply(c_scale(E(estimated_bipartite)$weight / max(E(estimated_bipartite)$weight)), 1,
                                     function(x) rgb(x[1]/255, x[2]/255, x[3]/255))
E(bipartite)$color = apply(c_scale(E(bipartite)$weight / max(E(bipartite)$weight)), 1,
                           function(x) rgb(x[1]/255, x[2]/255, x[3]/255))
la = la[, c(2, 1)]
# Plot networks: true and estimated
plot(bipartite, layout = la,
     vertex.color = c("red","black")[V(bipartite)$type + 1],
     vertex.shape = c("square", "circle")[V(bipartite)$type + 1],
     vertex.label = NA, vertex.size = 5)
plot(estimated_bipartite, layout = la,
     vertex.color = c("red","black")[V(estimated_bipartite)$type + 1],
     vertex.shape = c("square", "circle")[V(estimated_bipartite)$type + 1],
     vertex.label = NA, vertex.size = 5)
```

True adjacency             |  Estimated adjacency
:-------------------------:|:------------------------:
![](block-bipartite_files/figure-markdown_github/unnamed-chunk-1-1.png) | ![](block-bipartite_files/figure-markdown_github/unnamed-chunk-1-2.png)

True graph                 |  Estimated graph
:-------------------------:|:-------------------------:
![](block-bipartite_files/figure-markdown_github/unnamed-chunk-1-3.png) | ![](block-bipartite_files/figure-markdown_github/unnamed-chunk-1-4.png)
