### Cybersecurity Vulnerability Prioritization: Predicting CVE Exploitation

**Author:** William Mendez


#### Executive summary

This project investigates whether publicly available vulnerability metadata can predict which CVEs will be exploited in the wild, using CISA's Known Exploited Vulnerabilities (KEV) catalog as ground truth. The analysis confirms that structured metadata contains useful predictive signal: our best model achieves 61.5% Recall @ Top 10%, compared to 35.3% for traditional CVSS severity-based prioritization. However, the model does not capture the full attacker decision process—approximately 40% of exploited vulnerabilities remain outside the top-priority tier, highlighting the limits of public-data-only approaches.

#### Rationale

Security teams face an overwhelming volume of newly disclosed vulnerabilities—over 30,000 CVEs in 2024 alone. Traditional prioritization relies on CVSS severity scores, but high severity does not correlate reliably with actual exploitation. Only ~4% of all CVEs are ever exploited in the wild. Without better prioritization, organizations either waste resources patching low-risk vulnerabilities or miss critical threats hiding in the noise.

This project does not attempt to replace production-grade systems like EPSS. Instead, it asks a more fundamental question: **how much predictive signal exists in public vulnerability descriptors alone?** The answer informs whether structured metadata deserves a role in remediation decision-making.

#### Research Question

How effectively can publicly available vulnerability characteristics predict whether a newly disclosed CVE will later be confirmed as exploited in the wild? Specifically: can structured vulnerability metadata improve remediation prioritization compared to traditional severity-based approaches?

#### Data Sources

| Source | Description | Records |
|--------|-------------|---------|
| [CISA KEV Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) | Confirmed exploited vulnerabilities (target label) | ~1,200 CVEs |
| [NIST NVD](https://nvd.nist.gov/) | Vulnerability metadata, CVSS scores, CWE classifications | 176,972 CVEs (2018-2024) |

#### Methodology

**Problem Type:** Supervised binary classification (`is_kev = 1` if exploited, `0` otherwise)

**Model Progression (as specified in original proposal):**

1. **Severity-Only Baseline:** CVSS base score alone (represents traditional prioritization)
2. **Structured Metadata Model:** Decomposed CVSS vector + CWE + publication timing (Logistic Regression with L1/L2)
3. **Enriched Model:** Added NLP keyword extraction, vendor prevalence, MITRE ATT&CK mapping

**Feature Engineering:**
- **Technical DNA (CVSS Decomposed):** Attack Vector, Privileges Required, User Interaction
- **Behavioral Intent (NLP Keywords):** Remote Code Execution, Privilege Escalation, Security Bypass patterns extracted via regex
- **Attacker ROI (Vendor Prevalence):** High-value targets (Microsoft, Google, Apple, Cisco)
- **Tactical Context (MITRE ATT&CK):** CWEs mapped to "Initial Access" techniques

**Evaluation Strategy:**
- **Temporal Split:** Train on 2018-2023, test on 2024 (no data leakage)
- **Primary Metric:** Precision-Recall AUC (handles class imbalance)
- **Secondary Metrics:** ROC-AUC, Recall @ Top K% (simulates fixed patch budget)
- **Class Imbalance:** ~0.4% positive class; used balanced class weights and `scale_pos_weight`

#### Results

**Research Question Answered:** Yes, structured metadata contains useful predictive signal that improves prioritization quality compared to severity-only approaches.

| Model | Recall @ Top 10% | ROC-AUC | PR-AUC |
|-------|------------------|---------|--------|
| **Baseline (CVSS Severity Only)** | 35.3% | 0.7609 | 0.0104 |
| Structured Metadata (Logistic Regression) | 37.8% | 0.7646 | 0.0175 |
| Deep NVD Features + NLP Keywords | 50.6% | 0.7944 | 0.0352 |
| **Final Clean Model (XGBoost, Leakage-Free)** | 53.2% | 0.8492 | 0.0525 |
| **Final Hybrid Model (TF-IDF + PCA + XGBoost)** | **61.5%** | **0.8774** | 0.0492 |

**Interpretation:**
- By reviewing only the **top 10%** of CVEs ranked by model probability, security teams can identify **61.5%** of vulnerabilities that will eventually be exploited—versus **35.3%** using CVSS severity alone.
- This represents a meaningful improvement in prioritization efficiency, but **does not eliminate risk**: ~40% of exploited CVEs fall outside the top tier.

**Temporal Validation (Stress Test):**

The model was trained on 2018-2023 data and tested on future cohorts:
- 2024 CVEs: 53.2% Recall @ 10%
- 2025 CVEs: 54.3% Recall @ 10%
- 2026 Q1 CVEs: 53.6% Recall @ 10%

The model maintains predictive power 2+ years forward, suggesting the learned patterns reflect durable attacker preferences rather than transient trends.

**Limitations:**
- Public metadata alone cannot capture zero-day dynamics, attacker intent, or environment-specific factors
- The model identifies statistical tendencies, not certainties
- KEV inclusion is a lagging indicator; some exploited CVEs may never appear in KEV

#### Next steps

1. Compare model predictions against EPSS scores as external benchmark
2. Analyze false negatives to understand what signals are missing from public data
3. Export final model as `.joblib` for operational testing
4. Develop non-technical executive summary for stakeholder communication
5. Explore ensemble approaches combining multiple model architectures

#### Outline of project

This project follows a structured analytical workflow:

1. **Data Acquisition** - Fetched CVE metadata from NIST NVD and exploitation labels from CISA KEV
2. **Data Cleaning & Preprocessing** - Handled missing values (median imputation for CVSS), validated data types, checked for duplicates
3. **Exploratory Data Analysis** - Analyzed class imbalance (~0.4% positive), feature distributions, temporal trends, and correlations with target
4. **Feature Engineering** - Created predictive signals from CVSS vector components, NLP keyword extraction, vendor prevalence flags, and MITRE ATT&CK mappings
5. **Baseline Modeling** - Established CVSS-only baseline (35.3% Recall@10%), then built Logistic Regression model with engineered features
6. **Evaluation** - Compared models using Recall@K%, PR-AUC, and ROC-AUC; validated with temporal train/test split

**Primary Notebook:**
- [00_Capstone_EDA_Submission.ipynb](00_Capstone_EDA_Submission.ipynb) - Complete analysis pipeline from data loading through baseline model evaluation

##### Contact and Further Information

**Email:** william.mendez@gmail.com

**Program:** UC Berkeley ML/AI Professional Certificate

**Repository:** https://github.com/willmen/berkeley-m201.git
