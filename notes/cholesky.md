---
title: Cholesky Decomposition
categories: [math]
---

Some notes on the Cholesky decomposition with a focus on the sparse
case.

# Theory TODO

Every positive definite (and some indefinite) matrix $$H$$ has a
factorization of the form:

$$H = L D L^T$$

where $$L$$ is lower-triangular with unit diagonal elements, and $$D$$
is diagonal (positive when $$H$$ is positive definite).

# Algorithm

The following algorithm computes the Cholesky decomposition of a
matrix $$H$$ *in-place*, using the diagonal blocks to store $$D$$. See
[Wikipedia](https://en.wikipedia.org/wiki/Cholesky_decomposition#Avoiding_taking_square_roots)
for more details. Note that the algorithm presented here works on the
*upper* diagonal of the original matrix.

## Factorization

<div class="algorithm" markdown="1">
  - for $$i = 1, \ldots n$$ 
    - for $$k = 1, \ldots i − 1$$  *$$k \in$$ processed*
      - $$H_{ii} \gets H_{ii} -  H_{ik} H_{kk} H_{ik}^T$$<br/>

    - for $$j = i + 1, \ldots n$$  *$$j \in$$ remaining*
      - for $$k = i − 1, \ldots 1$$ *$$k \in$$ processed*
	    - $$H_{ji} \gets H_{ji} - H_{jk} H_{kk} H_{ik}^T$$ *fill-in !*
      - $$H_{ji} \gets H_{ji} \inv{H_{ii}}$$ <br />
</div>


## Solving

We solve the system $$Hx = z$$ using the following algorithm, which
essentially performs two back-substitutions on triangular systems and
a diagonal scaling:

<div class="algorithm" markdown="1">
   - for $$i = 1, \ldots n$$
     - $$x_i \gets z_i$$ <br/>
     - for $$j = 1, \ldots i − 1$$  *$$j \in$$ processed*
       - $$x_i \gets x_i – H_{ij} x_j$$ <br/>


   - for $$i = n, \ldots 1$$ 
       - $$x_i \gets \inv{H_{ii}} x_i$$ <br/>
     - for $$j = i + 1, \ldots n$$ *$$j \in$$ processed*
       - $$x_i \gets x_i – H_{ji}^T x_j$$ <br/>
</div>
 

# Sparse Algorithm

When many blocks are zero in matrix $$H$$, the algorithm may be
simplified based on an undirected *graph* structure, where
an edge $$(i, j)$$ indicates a non-zero matrix block between
coordinates $$i$$ and $$j$$.

By traversing the (undirected) graph in a depth-first *postfix*
(children first) fashion, one can orient the graph as a *Directed
Acyclic Graph* (DAG) and number the vertices accordingly. Using such
numbering, every vertex has an index greater than all its predecessors
(children) according to the topological sort. Edges go from children
to parents.

In other words, processing nodes by *increasing* indices corresponds
to a **postfix** (*i.e* children first) traversal, while processing
nodes by *decreasing* indices corresponds to a **prefix** traversal
(parents first).

## Factorization

We process nodes in a postfix order so that at a given point of the
factorization, already processed nodes may only be children of the
current node:

<div class="algorithm" markdown="1">
  - for each vertex $$i$$ in **postfix** order:
    - for each **child** vertex $$k$$ of $$i$$ *otherwise $$H_{ik}$$ is zero*
      - $$H_{ii} \gets H_{ii} -  H_{ik} H_{kk} H_{ik}^T$$ <br />

    - for each **remaining** vertex $$j$$:
      - for each **common child** $$k$$ of $$i$$ and $$j$$:   *otherwise either $$H_{ik}$$ or $$H_{jk}$$ is zero*
	    - $$H_{ji} \gets H_{ji} - H_{jk} H_{kk} H_{ik}^T$$    *may add a new edge !*
    - for each **parent** vertex $$j$$ of $$i$$: *otherwise $$H_{ij}$$ is zero*
      - $$H_{ji} \gets  H_{ji} \inv{H_{ii}}$$ <br />
</div>

Should fill-in happend between $$i$$ and $$j$$, the added edge is oriented
such that $$i$$ becomes a new child of $$j$$, which does not alter the
topological ordering.


## Solving

This is a straightforward adaptation of the dense case:

<div class="algorithm" markdown="1">
   - for each vertex $$i$$ in **postfix** order:
     - $$x_i \gets z_i$$ <br />
     - for each **child** vertex $$j$$ of $$i$$:
       - $$x_i \gets x_i – H_{ij} x_j$$ <br />
   - for each vertex $$i$$ in **prefix** order:
       - $$x_i \gets \inv{H_{ii}} x_i$$ <br />
     - for each **parent** vertex $$j$$ of $$i$$:
       - $$x_i \gets x_i – H_{ji}^T x_j$$ <br/>
</div>


# Using Graph Edges

In practice, it is convenient to implement the algorithm by
associating matrix blocks $$H_{ij}$$ with edges $$(i, j)$$ in the
graph. We will assume that the initial set of edges only contains
edges corresponding to the lower triangular part of $$H$$.

The process of orienting the graph as a DAG will **reverse** some
edges in the graph, which corresponds to **transposing** their matrix
block. After the DAG is obtained, operations on predecessor/successor
vertices will be achieved through the in/out edge sets. Depending on
which matrix block is needed, some matrix transposing might be
required, for instance:

<div class="algorithm" markdown="1">
  - for each vertex $$i$$:
    - for each **child** vertex $$k$$ of $$i$$
      - do something with $$H_{ik}$$
</div>

becomes:

<div class="algorithm" markdown="1">
  - for each vertex $$i$$:
    - for each **in-edge** $$e$$ of $$i$$:
      - do something with $$H_e^T$$ *since $$H_e$$ stores $$H_{ki}$$*
</div>

The adapted algorithms for factorization and solving are given below.

## Factorization

<div class="algorithm" markdown="1">
  - for each vertex $$i$$ in **postfix** order:
    - for each **in-edge** $$e=(k, i)$$ of $$i$$:
      - $$H_{ii} \gets H_{ii} -  H_e^T H_{kk} H_e$$ <br />
  
    - for each **remaining** vertex $$j$$:
      - for each edge pair $$e =(k, i), f=(k, j)$$:   *$$k$$ is a common child to $$i$$ and $$j$$*
	    - $$H_{(i, j)} \gets H_{(i, j)} - H_e^T H_{kk} H_f$$  *may add a new edge $$(i, j)$$*
    - for each **out-edge** $$e = (i, j)$$ of $$i$$:
      - $$H_e \gets  \inv{H_{ii}} H_e$$ <br />
</div>

Note: the fill-in line has been transposed to ensure that added edges
will be further processed by the algorithm.

## Solving

<div class="algorithm" markdown="1">
   - for each vertex $$i$$ in **postfix** order:
     - $$x_i \gets z_i$$ <br />
     - for each **in-edge** $$e=(j, i)$$ of $$i$$:
       - $$x_i \gets x_i – H_e^T x_j$$ <br />
   - for each vertex $$i$$ in **prefix** order:
       - $$x_i \gets \inv{H_{ii}} x_i$$ <br />
     - for each **out-edge** $$e = (i, j)$$ of $$i$$:
       - $$x_i \gets x_i – H_e x_j$$ <br/>
</div>



# Acyclic Graphs

When the DAG forms a *tree*, which is the case for acyclic graphs,
there is never any common child between two vertices. Therefore, no
fill-in can ever happen, and the factor algorithm reduces to the
following:


## Linear-Time Factorization

<div class="algorithm" markdown="1">
  - for each vertex $$i$$ in **postfix** order:
    - for each **child** vertex $$k$$ of $$i$$:
      - $$H_{ii} \gets H_{ii} -  H_{ik} H_{kk} H_{ik}^T$$ <br />
    - for **the** parent vertex $$j$$ of $$i$$, if any:
      - $$H_{ji} \gets  H_{ji} \inv{H_{ii}}$$ <br />
</div>


It is easy to show that such factorization has $$O(n)$$
complexity. Similarly, the solve procedure will have linear-time
complexity with respect to the number of vertices.

Even when the adjacency graph contains cycles, it can be useful to
perform an *incomplete* factorization by computing a spanning tree and
using the algorithm above. The associated solve algorithm can then act
as a *preconditioner* for iterative methods.

# Incremental Tridiagonal Factorization

The graph of a tridiagonal matrix $$T_k = \block{\alpha_k, \beta_k}$$
is a line, hence a tree. We consider the last coordinate to be the
root of the tree, and get the following simple incremental algorithm:

$$
   \begin{align}
   l_1 &= 1 \\ 
   d_1 &= \alpha_1 \\
   \\
   
   l_k &= d_{k-1}^{-1} \beta_k \\
   d_k &= \alpha_k - l_{k}^2 d_{k-1} \\
   \end{align}
$$
   
   where the Cholesky factors are $$L_k = \mat{1 & & & \\ l_2 & 1 & & \\ &
   l_3 &1 & \\ & & \ldots & 1}$$ and $$D_k = \diag(d_1, \ldots, d_k)$$.

If we need to solve $$T_k x_k = b_k$$, we may express the incremental
solution as:
	
$$x_k = L_k^{-T}y_k$$

in order to obtain the following on $$y_k$$:

$$D_k y_k = L_k^{-1} b_k = c_k$$
   
The last coordinate of $$c_k$$ can be computed easily from $$c_{k-1}$$ and
	$$b_k$$ by:

$$c_k^{(k)} + l_k c_{k-1}^{(k-1)} = b_k^{(k)}$$
   
The last coordinate of the solution vector $$x_k$$ can be obtained
from $$y_k$$:

$$x_k^{(k)} + l_k x_{k-1}^{(k-1)} = y_k^{(k)}$$

This procedure is used in the Lanczos formulation of the Conjugate
Gradient algorithm.
