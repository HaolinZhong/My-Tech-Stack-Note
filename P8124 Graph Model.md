# Intro



## Basics



- `model`  在概率/统计建模中, 模型就是在数据背后, **产生该数据的一系列概率分布的集合**.
- `independence constraint`
  - Graphical models are multivariate statistical models defined by independence constraints.
  - Joint distribution can be disastrously complex and has an enormous sample space.
  - **Conditional independence assumptions** are represented with graphical models.

- `Graphical Models`
  - A graphical model is a pair $(\mathcal{G}, \mathcal{P})$, where $\mathcal{G}$ is a graph and $\mathcal{P}$ is a set of distributions which factorizes according to $\mathcal{G}$.
  - A graph $\mathcal{G} = (V, E)$, where V is a set of vertices and E is a set of edges.
    - vertices index some R.V.
    - properties: ordered / unordered; cyclic / acyclic
- `causal graphical model`  a graphical model + some causal interpretation.





## Conditional Independence



- For **densities conditional independence** corresponds to: $p\left(x_1, x_2 \mid z\right)=p\left(x_1 \mid z\right) p\left(x_2 \mid z\right)$
  or equivalently $p\left(x_1 \mid x_2, z\right)=p\left(x_1 \mid z\right)$. (almost surely wrt $P$ )
  - Notation: $X_1 \perp X_2 \mid Z$.
  - if $Z$ is empty:  $X_1 \perp X_2 $; marginally conditional independence



- Properties:
  - `symmetry` $X_1 \perp X_2\left|Z \Longrightarrow X_2 \perp X_1\right| Z$
  - `decomposition` $X_1 \perp\left(X_2, X_3\right)\left|Z \Longrightarrow X_1 \perp X_2\right| Z$ 
  - `weak union` $X_1 \perp\left(X_2, X_3\right)\left|Z \Longrightarrow X_1 \perp X_2\right|\left(Z, X_3\right)$
  - `contraction` $X_1 \perp X_2\left|\left(Z, X_3\right), \space X_1 \perp X_3\right| Z \Longrightarrow X_1 \perp\left(X_2, X_3\right) \mid Z$ 
- and for positive distributions, also:
  - `intersection` $X_1 \perp X_2\left|\left(Z, X_3\right), \space X_1 \perp X_3\right|\left(Z, X_2\right) \Longrightarrow X_1 \perp\left(X_2, X_3\right) \mid Z$



- conditional indepence $\implies$ $\rho = 0$; 
- $\rho = 0$ $\not \implies$   conditional independence





# Directed Graph Models



