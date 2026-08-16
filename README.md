# Quantum Kernel $k$-Means for Credit-Card Fraud Detection

Reproduction notebooks for the paper *Quantum Kernel $k$-Means for
Credit-Card Fraud Detection: A Controlled Benchmark on Real Transaction Data*.

**Result:** under a budget-matched, held-out protocol, quantum kernel
$k$-means shows **no robust advantage** over tuned classical kernel clustering
on real credit-card fraud data. The one statistically significant advantage we
found ($p = 0.013$, at 8 qubits) is fully explained by how many feature-map
configurations we were permitted to search.

---

## The notebooks

Every notebook is **self-contained**: the batched statevector simulator, the
seven feature-map families and kernel $k$-means are all defined inline. There
is no package to install beyond pip dependencies, and nothing to import from
elsewhere in this repository.

Open any of them in Jupyter or Colab and run top to bottom.

| Notebook | What it does | Runtime |
|---|---|---|
| [`01_verify_method.ipynb`](notebooks/01_verify_method.ipynb) | Checks the inversion test returns the true overlap, that the batched simulator matches Qiskit exactly, and that kernel $k$-means reduces to $k$-means for a linear kernel. Prints PASS/FAIL. | ~2 min |
| [`02_reproduce_main_result.ipynb`](notebooks/02_reproduce_main_result.ipynb) | Re-runs selection and held-out scoring from scratch, then shows the full published table | ~4 min |
| [`03_search_budget_ablation.ipynb`](notebooks/03_search_budget_ablation.ipynb) | The central methodological result | ~1 min |
| [`04_imbalance_and_anomaly.ipynb`](notebooks/04_imbalance_and_anomaly.ipynb) | Why clustering fails at realistic base rates, and what works instead | ~2 min |
| [`05_all_paper_figures.ipynb`](notebooks/05_all_paper_figures.ipynb) | Regenerates all eight paper figures | ~1 min |

**Start with notebook 1** if you want to satisfy yourself the method is
implemented correctly, or **notebook 3** if you only care about the headline
argument.

---

## Method in one paragraph

Each transaction is encoded into an $n$-qubit state by a parameterised circuit
$U(\mathbf{x})$. Similarity is the state overlap
$K(\mathbf{x},\mathbf{y}) = |\langle\psi_\mathbf{y}|\psi_\mathbf{x}\rangle|^2$,
measured with the **inversion test**: run $U(\mathbf{x})$ forwards, then
$U(\mathbf{y})^\dagger$ backwards, and count the shots returning to all zeros.
That probability *is* the overlap — $n$ qubits, no ancilla, no controlled-SWAP.
The resulting Gram matrix is handed to kernel $k$-means, where the centroid
stays implicit in Hilbert space.

---

## Results

Held-out adjusted Rand index, mean over 30 independent resamples:

| $n$ | $k$-means | classical kernel | quantum kernel | $\Delta$ | $p$ |
|---|---|---|---|---|---|
| 2 | 0.770 | 0.829 | 0.820 | −0.0088 | 0.117 |
| 4 | 0.737 | 0.863 | 0.851 | −0.0119 | 0.025 |
| 8 | 0.686 | 0.858 | **0.867** | +0.0086 | 0.013 |
| 12 | 0.637 | 0.852 | 0.855 | +0.0026 | 0.422 |

The sign of the difference changes with register size and no magnitude exceeds
0.013 ARI. More qubits did not help.

**Search-budget ablation** at 8 qubits — the margin is a function of how many
configurations the quantum side may try:

| budget | 1 | 10 | 20 | 50 | 100 | 264 |
|---|---|---|---|---|---|---|
| mean margin | −0.390 | −0.006 | −0.002 | +0.002 | +0.005 | +0.009 |
| % of draws quantum leads | 2% | 19% | 34% | 67% | 91% | 100% |

The classical baseline was given 105 configurations. At comparable budgets the
two methods are indistinguishable.

---

## Protocol

The methodological core, and the part we would most like others to copy:

- **Development resamples** (seeds 0–3) choose hyperparameters — for *both*
  sides.
- **Held-out resamples** (30 draws, seeds 100–129) are used only for
  reporting, and are never consulted during selection.
- **Matched search budgets**: 264 quantum feature-map configurations against
  105 classical kernel configurations.
- **Paired statistics** across resamples, with confidence intervals.

---

## Data

The [ULB / Worldline credit-card fraud dataset](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud)
(284,807 transactions, 492 frauds, 0.173%). The notebooks download it at run
time from OpenML (ID 1597) — no Kaggle account needed, and this repository does
not redistribute it.

The data is under the **Database Contents License (DbCL) v1.0** and remains so;
the MIT licence here covers our code only. If you use it, credit Worldline and
the Machine Learning Group of Université Libre de Bruxelles, and cite
Dal Pozzolo et al., IEEE SSCI (2015).

`results/` holds the stored JSON output of the full experiments, which the
notebooks read for the published numbers.

---

## Licence

Code MIT (see `LICENSE`). Figures and manuscript text CC BY 4.0. Data as above.
