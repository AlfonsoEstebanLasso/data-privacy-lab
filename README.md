# Data Privacy Lab — Re-identification Attacks, Anonymization and Pseudonymization

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)
[![Python 3](https://img.shields.io/badge/Python-3-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=flat-square&logo=jupyter&logoColor=white)](https://jupyter.org/)
[![pandas](https://img.shields.io/badge/pandas-150458?style=flat-square&logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=flat-square&logo=scikitlearn&logoColor=white)](https://scikit-learn.org/)
[![pycryptodome](https://img.shields.io/badge/pycryptodome-2C7FB8?style=flat-square)](https://www.pycryptodome.org/)

**First break a naively "anonymized" data release with a linkage attack, then defend it — and measure what each defense still leaks 🔐**

Coursework project — BSc in Applied Data Science, Universitat Oberta de Catalunya (UOC), Data Privacy and Security course.

Two hands-on notebooks that first **break** a naively "anonymized" data release and then **protect** it, using a worker-absenteeism scenario: an HR department publishes a demographics table and a detailed absence-records table and assumes that removing names from one of them is enough.

> Exercise headers were rewritten in English for this repository; the author's analysis and justification cells keep their original Spanish narrative. Notebook narrative is in Spanish.

## Objective

Show, with working code on real and synthetic microdata, why removing direct identifiers does not anonymize a dataset, and demonstrate the standard countermeasures: generalization, MDAV microaggregation, additive noise with a measured utility cost, salted cryptographic pseudonymization, and encryption of sensitive attributes — including what each encryption scheme still leaks to an attacker.

## Data & methods

| File | Description |
|---|---|
| `data/absenteeism.csv` | [Absenteeism at work](https://archive.ics.uci.edu/dataset/445/absenteeism+at+work) dataset, UCI Machine Learning Repository (Martiniano, Ferreira & Ferreira, 2018), licensed CC BY 4.0. The copy here is the course-formatted version (semicolon-separated). |
| `data/workers.csv` | Synthetic 23-row worker table created for the course exercise. All names and attributes are fictional; it contains no real personal data. |

**`01_reidentification_attack.ipynb`**
- Linkage attack crossing the two "anonymized" tables by matching total absences against summed absence hours per ID.
- Targeted re-identification of a single person from an auxiliary fact (shortest commute).
- Identifier vs. quasi-identifier analysis and removal.
- Generalization of `age` into 10-year intervals.
- Microaggregation of `age` with MDAV (k = 3), preserving the mean (39.5 vs. 39.4 after anonymization) while blocking the earlier re-identification.
- Uncorrelated additive Gaussian noise on a quasi-identifier, with the utility loss measured as MSE and a sweep to find the largest noise level keeping MSE < 0.02.

**`02_pseudonymization_crypto.ipynb`**
- Pseudonymization of identifiers with a double hash (RIPEMD-160 then SHA-512, via `pycryptodome` and `hashlib`).
- Attacks against it, with timing: brute force without knowledge of the hash chain (fails) vs. a dictionary attack knowing it, which recovers a known person's record in milliseconds; a common-names dictionary attack recovers records even when the attacker is unsure which of two hash chains was used, at double the hash cost.
- The fix: per-record random salts, with both attacks re-run to show they no longer succeed.
- Encryption of the sensitive `absences` attribute with three cryptosystems — RSA (textbook, deterministic), ElGamal (probabilistic) and the A5/1 stream cipher implemented from its shift registers — followed by a ciphertext-only analysis of what each scheme still leaks: deterministic RSA reveals record lengths and equality between records (and falls to a plaintext dictionary attack over a small message space), ElGamal hides equality but not counts, A5/1 hides the record structure entirely.
- An independence (fairness) check of the predicted `insurance` attribute with respect to the protected attribute `sex`, using a chi-squared test.

## Why this matters for biomedical data

Absence records are health-related information, which is exactly the kind of attribute that clinical and omics datasets carry. The techniques exercised here — quasi-identifier analysis, k-anonymity-style generalization, microaggregation, salted pseudonymization and encryption with known leakage properties — are the standard toolbox for sharing patient-level data under regulations such as the GDPR.

## Tech stack

Python, pandas, NumPy, scikit-learn (KMeans for MDAV), matplotlib, SciPy (chi-squared test), pycryptodome (RIPEMD-160, ElGamal utilities), hashlib.

## How to run

The notebooks were authored and executed in Google Colab and keep their original outputs; the first cells mount Google Drive and read the course folder layout. To run locally:

```bash
pip install -r requirements.txt
jupyter lab
```

then replace the Drive-mount cells with a local path pointing at `data/` (the CSVs are semicolon-separated; notebook 02 already reads them with `sep=';'`). Notebook 01 was originally executed against a slightly different formatting of the same tables (comma-separated, `absences` as a total count), so re-running it against `data/` requires minor adaptation of its two load cells.

## Repository structure

```
data-privacy-lab/
├── 01_reidentification_attack.ipynb   # linkage attack, MDAV, generalization, additive noise
├── 02_pseudonymization_crypto.ipynb   # double-hash pseudonymization, attacks, RSA/ElGamal/A5-1
├── data/
│   ├── absenteeism.csv                # UCI "Absenteeism at work" (CC BY 4.0)
│   └── workers.csv                    # synthetic course dataset (fictional)
├── requirements.txt
└── README.md
```
