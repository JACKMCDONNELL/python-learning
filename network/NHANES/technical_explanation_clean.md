# Biovista Technical Appendix

This document describes the mathematical foundations behind the Biovista network model.

---

# 1. Data Harmonization (Standardization)

Biomarkers are standardized before modeling so each variable contributes on the same scale.

$$
z_{ij} = \frac{x_{ij} - \mu_j}{\sigma_j}
$$

Where:

- \( x_{ij} \) = observed value  
- \( \mu_j \) = NHANES population mean  
- \( \sigma_j \) = NHANES population SD  

---

# 2. KNN Missing Data Imputation

Biovista uses K-Nearest Neighbors (KNN) to fill missing biomarker values.

## 2.1 Distance Between Individuals

Let \( \mathcal{O}_i \) be the set of variables observed for person \( i \).  
Distance between individuals \( i \) and \( k \):

$$
d(i,k)
=
\sqrt{
\sum_{l \in (\mathcal{O}_i \cap \mathcal{O}_k)}
(x_{il} - x_{kl})^2
}
$$

Only variables observed in *both* individuals contribute.

---

## 2.2 KNN Imputation Rule

The imputed value for a missing biomarker \( j \) in person \( i \):

$$
\hat{x}_{ij}
=
\frac{1}{K}
\sum_{k \in N_K(i,j)} x_{kj}
$$

Where:

$$
N_K(i,j)
=
\left\{
k \;\middle|\;
k \text{ is one of the } K \text{ nearest neighbors of } i
\text{ with an observed value for variable } j
\right\}
$$

We use:

- \( K = 5 \)  
- Euclidean distance  
- Standardized features  

---

# 3. Gaussian Graphical Model (GGM)

## 3.1 Covariance Matrix

Let \( Z \) be the standardized data matrix.  
The empirical covariance is:

$$
S = \frac{1}{n} Z^\top Z
$$

---

## 3.2 Graphical Lasso Objective

We estimate the precision matrix \( \Theta \) using a sparse penalized likelihood:

$$
\Theta^\* =
\arg\min_{\Theta \succ 0}
\Big[
\mathrm{tr}(S\Theta)
-
\log\det(\Theta)
+
\lambda \lVert \Theta \rVert_{1}
\Big]
$$

Where:

$$
\lVert \Theta \rVert_1 = \sum_{i \ne j} |\Theta_{ij}|
$$

The penalty forces many entries of \( \Theta \) to zero, producing a sparse network.

---

## 3.3 Partial Correlations

From the estimated precision matrix:

$$
\rho_{ij}
=
-
\frac{
\Theta_{ij}
}{
\sqrt{
\Theta_{ii}\Theta_{jj}
}
}
$$

With:

$$
\rho_{ii} = 0
$$

Edges are kept when:

$$
|\rho_{ij}| \ge 0.10
$$

---

# 4. Node Attributes

For each biomarker node \( j \):

$$
\text{NodeZ}(j) = z_j
$$

$$
\text{System}(j) = \text{assigned physiological system}
$$

Nodes are colored by z-score and outlined by system.

---

# 5. Patient-Level Embedding

Each patient is projected onto the same physiologic network.

$$
z_j = \frac{x_j - \mu_j}{\sigma_j}
$$

- Node color = direction/magnitude of abnormality  
- Node size = \(|z_j|\)  
- Edges = fixed partial correlations from population GGM  

---

# 6. Network Deterioration Score

## 6.1 Node Contribution

Let:

- \( z_j \) = patient z-score  
- \( d_j \) = degree (number of edges of node \( j \))

Then each biomarker contributes:

$$
s_j = |z_j| \cdot d_j
$$

---

## 6.2 Total Network Score

$$
S_{\text{network}}
=
\sum_{j=1}^{p}
|z_j| \cdot d_j
$$

Interpretation:

- Low score → well-regulated physiology  
- High score → multi-system dysregulation  

Examples:

$$
S_{\text{Michael}} = 0
$$

$$
S_{\text{Daniel}} = 38.5
$$

---

# 7. Force-Directed Layout

Graph layout uses a force-based system:

$$
F = F_{\text{attraction}} + F_{\text{repulsion}}
$$

- Connected nodes pull together  
- All nodes repel  

This reveals physiologic clusters visually.

---

# 8. Software Implementation

Biovista prototype uses:

- **pandas** — data handling  
- **scikit-learn**  
  - `KNNImputer`  
  - `StandardScaler`  
  - `GraphicalLasso`  
- **networkx** — graph structure  
- **matplotlib** — visualization  

