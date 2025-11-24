# **Biovista Technical Appendix (Mathematical & Computational Framework)**

## **1. Overview**

Biovista integrates high-dimensional clinical and physiologic data to generate:

1. **Imputed, harmonized biomarker matrices**
2. **Standardized z-scores**
3. **A Gaussian Graphical Model (GGM) of partial correlations**
4. **A patient-specific projection onto this population network**
5. **A Network Stress Score (NSS)** representing systemic dysregulation

This appendix describes the underlying mathematics used in the pipeline.

---

# **2. Data Harmonization & Imputation**

Biovista takes heterogeneous NHANES biomarkers and combines them into a unified matrix. Missing values are handled using **K-Nearest Neighbors (KNN) imputation**.

### **2.1 Distance Function**

For an observation ( i ) and a potential neighbor ( k ):

[
\displaystyle
d(i,k) =
\sqrt{
\sum_{l \in \mathcal{O}*i \cap \mathcal{O}*k}
\left( x*{il} - x*{kl} \right)^2
}
]

Where:

* ( \mathcal{O}_i ) = set of variables observed for subject ( i )
* Only variables observed in **both** subjects contribute to the distance

### **2.2 KNN Imputation Rule**

[
\displaystyle
\hat{x}*{ij} =
\frac{1}{K}
\sum*{k \in N_K(i,j)}
x_{kj}
]

Where (N_K(i,j)) is the set of (K) nearest neighbors with non-missing values for variable (j.)

We use (K = 5).

---

# **3. Standardization (Z-Scoring)**

Each variable is standardized to have population mean 0 and standard deviation 1:

[
\displaystyle
Z_{ij}
= \frac{ \hat{x}_{ij} - \mu_j }{ \sigma_j }
]

Where:

* ( \hat{x}_{ij} ) = imputed value
* ( \mu_j ) = population mean for variable (j)
* ( \sigma_j ) = population standard deviation

This step makes all biomarkers comparable (unitless).

---

# **4. Gaussian Graphical Model (GGM)**

Biovista estimates a **sparse inverse covariance matrix** using the **Graphical Lasso**, identifying conditional dependencies between physiological variables.

Let:

* ( S ) = sample covariance matrix
* ( \Theta = \Sigma^{-1} ) = precision matrix

### **4.1 Optimization Objective**

Graphical Lasso solves:

[
\begin{aligned}
\hat{\Theta} =
\arg\min_{\Theta \succ 0}
\Big{
& \mathrm{trace}(S\Theta)

* \log \det(\Theta) \
  & \quad + \lambda \sum_{j \neq k}
  |\Theta_{jk}|
  \Big}
  \end{aligned}
  ]

Where:

* ( \lambda ) controls sparsity
* Off-diagonal zeros indicate **no conditional dependency**

### **4.2 Partial Correlations**

After estimating ( \hat{\Theta} ):

[
\displaystyle
\rho_{jk} =

* \frac{ \hat{\theta}*{jk} }
  { \sqrt{ \hat{\theta}*{jj} \hat{\theta}_{kk} } }
  ]

Interpretation:

* ( \rho_{jk} > 0 ): direct positive relationship controlling for all others
* ( \rho_{jk} < 0 ): direct inverse relationship
* ( \rho_{jk} = 0 ): conditionally independent

Edges with (|\rho| < 0.10) are removed for clarity.

---

# **5. Patient-Level Projection**

The GGM defines the structure of the physiologic network. A patient is then projected onto this network by mapping their **individual z-scores** to node values.

Let:

* ( z_j ) = patient’s z-score for biomarker (j)
* ( E = ) set of edges in the network

---

# **6. Network Stress Score (NSS)**

Biovista creates a single interpretable index summarizing system-level dysregulation.

### **6.1 Node Stress**

Each variable contributes stress proportional to:

* How abnormal it is (|z|)
* How central it is in the physiologic network (degree)

[
\displaystyle
s_j = |z_j| \cdot \deg(j)
]

### **6.2 Total Network Score**

[
\displaystyle
\text{NSS}
= \sum_{j=1}^{p} s_j
]

Where:

* (p) = number of variables in the patient’s network
* Higher scores = more global physiologic perturbation

---

# **7. Interpretation of NSS**

* **Low NSS (~0–10)**

  * "Michael" example: normal biomarkers → minimal network stress
* **Moderate NSS (~15–30)**

  * Local dysregulation but retains network stability
* **High NSS (30+)**

  * "Daniel" example: multi-system abnormalities amplify network stress
  * Nonlinear propagation via hubs magnifies pathology

---

# **8. Clinical Meaning**

While NSS is not a diagnostic test, it provides:

1. **A systems-biology view** of patient health
2. **A graphical explanation** of how abnormalities cluster
3. **A way to quantify diffuse metabolic / lifestyle risk**
4. **A tool for monitoring improvement over time**

---

# **9. Summary**

Biovista combines:

* KNN imputation
* Standardization
* Graphical Lasso
* Partial correlations
* Network theory
* Patient z-score overlays

…into a clinically interpretable framework for understanding systemic physiologic health.

---

If you’d like, I can also generate:

* **LaTeX PDF version**
* **DOCX Word version** (with equations fully rendered)
* **A slide-ready appendix**
* **A GitHub-ready `README.md`**

Just tell me!
