# **Biovista Technical Appendix**

# **1. Data Harmonization**

Biovista standardizes all biomarkers before modeling.

## **1.1 Standardization Formula**

$$
z_{ij} = \frac{x_{ij} - \mu_j}{\sigma_j}
$$

Where:

$$
x_{ij} = \text{value for person } i \text{ on variable } j
$$

$$
\mu_j = \text{NHANES mean of variable } j
$$

$$
\sigma_j = \text{NHANES standard deviation of variable } j
$$

---

# **2. KNN Missing Data Imputation**

## **2.1 Distance Between Two Individuals**

$$
d(i,k)
=
\sqrt{
\sum_{l \in \mathcal{O}_i \cap \mathcal{O}_k}
\left( x_{il} - x_{kl} \right)^2
}
$$


Where:

$$
\mathcal{O}_i =
\text{set of variables observed for individual } i
$$

$$
\mathcal{O}_k =
\text{set of variables observed for individual } k
$$

Only overlapping observed variables contribute to the distance.

---

## **2.2 KNN Imputation Rule**

## **2.2 KNN Imputation Rule**

Given a partially observed matrix of biomarkers, missing values are filled using **K-Nearest Neighbors (KNN) imputation**.

The imputed value for variable \( j \) of individual \( i \) is:

$$
\hat{x}_{ij}
=
\frac{1}{K}
\sum_{k \in N_K(i,j)}
x_{kj}
$$

Where:

$$
N_K(i,j)
=
\left\{
k \,\middle|\,
k \text{ is one of the } K \text{ nearest neighbors of individual } i
\text{ with an observed value for } x_{kj}
\right\}
$$

Distance between subjects is computed only on the variables observed in common:

$$
d(i,k) =
\sqrt{
\sum_{l \in \mathcal{O}_i \cap \mathcal{O}_k}
\left( x_{il} - x_{kl} \right)^2
}
$$

And we use:

- \( K = 5 \)
- Euclidean distance  
- Standardized variables so no feature dominates


---

# **3. Gaussian Graphical Model (GGM)**

## **3.1 Covariance Estimate**

Let (Z) be the standardized matrix.
The empirical covariance matrix is:

$$
S = \frac{1}{n} Z^\top Z
$$

---
## **3.2 Graphical Lasso Objective**

The precision matrix \( \Theta \) is estimated by solving the penalized likelihood:

$$
\Theta^\* =
\arg\min_{\Theta \succ 0}
\left[
\mathrm{tr}(S \Theta)
-
\ln \det(\Theta)
+
\lambda \lVert \Theta \rVert_{1}
\right]
$$

Where:

- \( \Theta = \Sigma^{-1} \) is the precision matrix  
- \( \lambda \) is the sparsity penalty  
- The \( \ell_{1} \) norm is:

$$
\lVert \Theta \rVert_1
=
\sum_{i \ne j} \lvert \Theta_{ij} \rvert
$$

The penalty forces many off-diagonal elements to zero → producing a **sparse**, **interpretable** physiological network structure.


## **3.3 Partial Correlations**

For each pair of biomarkers (i) and (j):

$$
\rho_{ij}
=
-\frac{
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

Edges in the network are kept if:

$$
|\rho_{ij}| \ge 0.10
$$

---

# **4. Node Attributes in the Network**

For each node representing a biomarker:

$$
\text{NodeZ}(j) = z_j
$$

$$
\text{System}(j)
=
\text{assigned physiology category}
$$

---

# **5. Patient-Level Embedding**

Each patient’s biomarker panel is mapped onto the network.

The per-node value for a patient with observed (x_j):

$$
z_j
=
\frac{x_j - \mu_j}{\sigma_j}
$$

Node color reflects (z_j).
Node size reflects (|z_j|).
Edges reflect partial correlations from the NHANES population.

---

# **6. Network Deterioration Score**

## **6.1 Node Contribution**

Each biomarker contributes a node score:

$$
s_j = |z_j| \cdot d_j
$$

Where:

$$
d_j = \text{degree of node } j \text{ (number of edges)}
$$

---

## **6.2 Total Network Score**

The patient’s overall network dysregulation is:

$$
S_{\text{network}}
=
\sum_{j=1}^{p}
|z_j| \cdot d_j
$$

Interpretation thresholds:

$$
S \approx 0
\quad\rightarrow\quad
\text{healthy, well-regulated}
$$

$$
10 \le S \le 20
\quad\rightarrow\quad
\text{moderate physiologic strain}
$$

$$
S \ge 30
\quad\rightarrow\quad
\text{broad multisystem dysregulation}
$$

Example scores:

$$
S_{\text{Michael}} = 0
$$

$$
S_{\text{Daniel}} = 38.5
$$

---

# **7. Visualization Layout**

Biovista uses a force-directed network, approximated by:

$$
F = F_{\text{attraction}} + F_{\text{repulsion}}
$$

Attraction pulls connected nodes together.
Repulsion spreads nodes apart.

This reveals dysregulated clusters visually.

