---
title: Projective Geometry
categories: [math]
---

# Projective Space

Consider non-zero elements of a real vector space $$E\simeq\RR^n$$,
equipped with the *colinearity* equivalence relation:

$$x \sim y \iff \exists \lambda \in \RR^\star,\quad x = \lambda y$$

The projective space $$\PS(E)$$ is the quotient space for $$\sim$$. In
 other words, $$\PS(E)$$ is the space of vector lines of $$E$$. The
 *dimension* of the projective space $$\PS(E)$$ is $$n-1$$. In
 particular:
 
 - the projective *line* is $$\PS\block{\RR^2}$$
 - the projective *plane* is $$\PS\block{\RR^3}$$
 
Generally, we will denote  $$\PS^n=\PS\block{\RR^{n+1}}$$.

# Affine Decomposition

Consider an affine hyperplane $$H$$ of $$E$$ that does not contain the
origin. Then, any vector line of $$E$$ that is not parallel to $$H$$
(*i.e.* vector lines of $$\vec{H}$$) can be uniquely associated with a
point of $$H$$: the point where the line intersects $$H$$. The
remaining vector lines are parallel to $$H$$ and are elements of
$$\PS\block{\vec{H}}$$, by definition. We obtain the following
decomposition of projective spaces:

$$\PS\block{E} = H \cup \PS\block{\vec{H}}$$

Equivalently, up to isomorphism:

$$\PS^n \simeq \RR^n \cup \PS^{n-1}$$

# Homogeneous Coordinates

For $$E\simeq\RR^n$$, consider the affine hyperplane where the last
coordinate is $$1$$:

$$H=\{x \in E, \ e_n^Tx = 1\}$$

where $$e_n = (0, \dots, 1)^T$$ is the $$n$$-th canonical basis
vector. In particular, the direction space $$\vec{H} = \{x\in E,\ x_n
= 0\}$$ is called the *points at infinity*, and any vector line not in
$$\vec{H}$$ corresponds to a unique point of $$H$$, obtained by
dividing by the last coordinate.


# Projective Frame

Given $$n$$ linearly independent lines in $$E=\RR^n$$, one can extract a basis
$$\block{e_i}_i$$ of $$E$$ by picking an *arbitrary* non-zero vector on each
line. Given this basis, one can reconstruct any vector from its coordinates, and
therefore any vector line. However, it is important that the reconstructed line
does not depend on which basis vector was chosen to represent each line. For
instance, given a line $$\lambda v$$, one may choose $$v$$ or $$2v$$ as a
representant, which would change the reconstructed vector (assuming fixed
coordinates), thus affecting the reconstructed line as well. In other words, a
system of $$n$$ lines is insufficient to reconstruct any other given line: we
need more information.

Formally, given coordinates $$x_i$$ and a basis $$\block{e_i}_i$$, a
reconstructed vector is $$x = \sum_i x_i \lambda_i e_i$$ and we would like to
make sure that changing any $$\lambda_i$$ produces a result colinear with $$x$$:

$$\sum_i x_i \lambda_i e_i \sim \sum_i x_i e_i$$

In block notation, if we assemble the $$\block{\lambda_i}_i$$ into a diagonal
matrix $$L=\diag\block{\lambda_i}$$ and the basis vectors into a matrix $$B$$,
we require that there exists some non-zero $$\lambda$$ such that, for all
non-zero $$x$$:

$$BLx = \lambda Bx$$

Since the above must hold for all non-zero $$x$$ and $$B$$ is invertible, the
only solution is to have all $$\lambda_i$$ equal to $$\lambda$$. It can be
checked that the condition:

$$\forall x \neq 0 \quad Lx = \lambda x$$

is equivalent to:

$$L\ones = \lambda \ones$$

and it suffices to enforce our colinearity condition on $$B\ones$$. Therefore,
we need to somehow store the direction of $$B\ones$$ alongside $$BL$$ in order
to "calibrate" $$L$$. Conversely, given any set of $$n$$ lines $$BL$$ and a
direction vector $$d$$ for $$B\ones$$, one can recalibrate matrix $$L$$ as above
to enforce the colinearity condition. It is then possible to recover any vector
line from its $$\block{x_i}_i$$, called the *projective coordinates*.

# Homographies

# Duality
