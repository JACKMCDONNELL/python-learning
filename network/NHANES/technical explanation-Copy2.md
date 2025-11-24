# **Biovista Technical Appendix**

# **1. Data Harmonization**

Biovista standardizes all biomarkers before modeling.

## **1.1 Standardization Formula**

$$
z_{ij} = \frac{x_{ij} - \mu_j}{\sigma_j}
$$

Where:

- \( x_{ij} \) = value of variable \( j \) for person \( i \)  
- \( \mu_j \) = NHANES mean of variable \( j \)  
- \( \sigma_j \) = NHANES standard deviation  

---

# **2. KNN Missing Data Imputation**

## **2.1 Distance Between Two Individuals**

The KNN distance between two individuals \( i \) and \( k \) is:

$$
d(i,k)=
\sqrt{
\sum_{l \in \mathcal{O}_i \cap \mathcal{O}_k}
\left( x_{il} - x_{kl} \right)^2
}
$$

Where:

- \( \mathcal{O}_i \): variables observed for individual \( i \)  
- \( \mathcal{O}_k \): variables observed for individual \( k \)  

---

## **2.2 KNN Imputation Rule**

The imputed value for variable \( j \) for subject \( i \) is:

$$
\hat{x}_{ij} = 
\frac{1}{K}
\sum_{k \in N_K(i,j)} x_{kj}
$$

Where the neighbor set is:

$$
N_K(i,j)=
\left\{
k \;\middle|\;
k \text{ is one of the } K \text{ nearest neighbors of } i 
\text{ with a known value for variable } j
\right\}
$$

The distance used is:

$$
d(i,k)=
\sqrt{
\sum_{l \in \mathcal{O}_i \cap \mathcal{O}_k}
\left( x_{il} - x_{kl} \right)^2
}
$$

Parameters:

- \( K=5 \)  
- Euclidean distance  
- Standardized variables before computing distances  

---

# **3. Gaussian Graphical Model (GGM)**

## **3.1 Covariance Estimate**

Let \( Z \) be the standardized data.  
Empirical covariance:

$$
S = \frac{1}{n} Z^\top Z
$$

---

## **3.2 Graphical Lasso Objective**

The precision matrix \( \Theta \) is estimated via:

$$
\Theta^\* =
\arg\min_{\Theta \succ 0}
\left[
\mathrm{tr}(S\Theta)
-
\ln\det(\Theta)
+
\lambda \lVert \Theta \rVert_1
\right]
$$

Where:

- \( \Theta = \Sigma^{-1} \)  
- \( \lambda \) = sparsity penalty  
- \( \lVert \Theta \rVert_1 = \sum_{i \ne j} |\Theta_{ij}| \)  

---

## **3.3 Partial Correlations**

Partial correlation between biomarkers \( i \) and \( j \):

$$
\rho_{ij} =
-\frac{
\Theta_{ij}
}{
\sqrt{\Theta_{ii}\Theta_{jj}}
}
$$

With:

$$
\rho_{ii} = 0
$$

Edges kept if:

$$
|\rho_{ij}| \ge 0.10
$$

---

# **4. Node Attributes**

Each biomarker node has:

- A Z-score \( z_j \)  
- A physiological-system category  
- A degree (number of edges)  

---

# **5. Patient-Level Projection**

A patient with measurement \( x_j \) receives:

$$
z_j = \frac{x_j - \mu_j}{\sigma_j}
$$

Nodes are then colored and sized according to \( z_j \).

---

# **6. Network Deterioration Score**

## **6.1 Per-Node Contribution**

$$
s_j = |z_j| \cdot d_j
$$

Where:

- \( d_j \) = degree of node \( j \)

---

## **6.2 Total Network Score**

$$
S_{\text{network}} = \sum_{j=1}^{p} |z_j| \cdot d_j
$$

Interpretation:

- \( S \approx 0 \): healthy  
- \( 10 \le S \le 20 \): moderate strain  
- \( S \ge 30 \): multisystem dysregulation  

Examples:

$$
S_{\text{Michael}} = 0
$$

$$
S_{\text{Daniel}} = 38.5
$$

---

# **7. Network Layout Physics**

Force-directed layout:

$$
F = F_{\text{attraction}} + F_{\text{repulsion}}
$$

This visually separates physiological clusters.


