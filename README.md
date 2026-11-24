# Three-Dimensional Detector-Graph Decoding

**Phenomenological noise, open-boundary matching, and likelihood-weighted corrections**

This repository investigates error syndromes and logical parity in a small three-dimensional detector graph. It includes an explicit binary fault-to-syndrome map, a minimum-weight perfect matching (MWPM) decoder with absorbing boundaries, Monte Carlo logical-failure estimates, and an extension to orientation-dependent fault probabilities.

The implementation is contained in a single Jupyter notebook. Numerical checks and plotted results are available directly in its executed cells; no auxiliary Python modules or data files are required.

> **Scope.** The lattice is a **phenomenological detector graph**, not a circuit-derived Raussendorf–Harrington–Goyal (RHG) photonic cluster state. The variable `distance` specifies a geometric lattice size, not a validated physical code distance. Neither the plots nor the illustrative decoder rates establish a fault-tolerance threshold.

## Repository

```text
topological-detector-graph-decoding/
├── README.md
└── Topological_Detector_Graph_Decoding.ipynb
```

## Mathematical model

### Detector geometry and logical parity

The base model uses a cubic graph with $d\times d\times d$ vertices. The planes $x=0$ and $x=d-1$ act as absorbing boundaries; the remaining vertices are detectors. A possible elementary fault is associated with each retained graph edge. Boundary-plane tangential edges are excluded.

With $m$ candidate faults and $r$ detectors, the binary incidence matrix $H\in\mathbb{F}_2^{r\times m}$ maps a fault configuration $\mathbf e\in\mathbb{F}_2^m$ to a syndrome:

$$
\mathbf s = H\mathbf e \pmod 2.
$$

A logical observable is defined by the parity of faults crossing the left boundary cut:

$$
o = \boldsymbol\ell^{\mathsf T}\mathbf e \pmod 2.
$$

A boundary-to-boundary fault chain can therefore have zero measured syndrome but nonzero logical parity. This supplies a controlled example of the ambiguity faced by syndrome-only decoders.

**Geometric spanning length.** The shortest fault chain joining the two absorbing
$x$-boundary planes has $d_{\mathrm{eff}}=d-1$ edges: it has length 2 for $d=3$
and length 4 for $d=5$. This is a property of the constructed detector graph,
**not** an experimentally validated RHG code distance. An even-length chain
can create a tie between corrections terminating at opposite boundaries.
At $d=3$, an exhaustive isolated-fault check finds that 9 of the 18 $x$-oriented
single-edge faults produce a logical-parity error under the fixed decoder's
boundary tie convention. This explains why the pilot can show leading-order
$O(p)$ logical failure at small fault probability. Odd geometric lengths
$d_{\mathrm{eff}}=3,5$ correspond to $d=4,6$ in this model.

### Uniform independent edge faults

In the base model, each candidate edge is faulty with independent probability $p$. The decoder looks for a correction $\hat{\mathbf e}$ satisfying

$$
H\hat{\mathbf e}=\mathbf s\pmod 2.
$$

For equal independent probabilities below $1/2$, unit edge weights give the same ordering of corrections as their relative negative log-likelihood, apart from shortest-path and matching degeneracies. NetworkX is used for shortest paths and MWPM on a small auxiliary matching graph with virtual boundary nodes.

For $M$ independent trials, the reported logical-failure estimate is

$$
\widehat p_L=\frac{1}{M}\sum_{k=1}^{M}
\mathbf 1\!\left[\boldsymbol\ell^{\mathsf T}
(\mathbf e_k\oplus\hat{\mathbf e}_k)=1\right].
$$

The notebook reports the plug-in binomial standard error. Its small pilot runs are not sufficient for threshold extraction or reliable rare-event estimates.

### Independent photon-presence baseline

A separate elementary calculation considers the probability that all $n$ photons are present when source, channel, and detection efficiencies are independent:

$$
P_{\mathrm{all\ present}}=(\eta_s\eta_c\eta_d)^n.
$$

This is a resource-availability estimate. It is **not** an erasure-decoding model and does not convert photon loss into the edge-error probabilities used by the matching decoder.

## Extension: nonuniform faults and likelihood-weighted decoding

The uniform decoder does not distinguish between fault mechanisms with different probabilities. The final notebook section introduces two independently assigned classes:

- $p_x$ for graph edges parallel to the $x$-axis;
- $p_{\perp}$ for edges transverse to the $x$-axis.

For each edge $e$ with $0<p_e<1/2$, the likelihood-derived correction weight is

$$
w_e=\log\left(\frac{1-p_e}{p_e}\right).
$$

The extension applies these weights to graph shortest paths and the auxiliary matching problem. **Both decoders use the same geometry, logical observable, and sampled fault configurations**, isolating the effect of weighting from changes in the data.

This addition improves the expressiveness of the assumed noise and decoding models. It does **not** provide a microscopic noise model of quantum-dot emitters, correlated photonic fusion errors, or a circuit-derived RHG detector error model.

### Interpretation of the comparison

The included demonstration uses `d=3`, `shots=120`, `p_x=0.06`, `p_transverse=0.015`, and seed `9`. In that finite sample, the observed logical-failure fractions are:

| Decoder | Observed failures / shots | Failure fraction |
|---|---:|---:|
| Uniform edge weights | 33 / 120 | 0.275 |
| Likelihood-derived edge weights | 38 / 120 | 0.317 |

The weighted decoder performs worse in **this particular short run**. Its use is motivated by the assumed likelihood model, not by a guaranteed finite-sample advantage. Equal-length boundary corrections and other degeneracies can also affect the comparison; a larger paired study would need to control tie conventions. The two estimates are paired because they come from identical sampled faults, and the modest sample size precludes a strong performance conclusion. Larger experiments, multiple seeds, and appropriate uncertainty treatment are needed before comparing decoder performance quantitatively.

## Notebook organization

| Section | Content |
|---|---|
| Part 1 | Graph-state and RHG context; distinction between physical architecture and detector graph |
| Part 2 | Stabilizers, binary syndromes, and logical observables |
| Part 3 | Cubic detector graph, boundaries, and incidence-matrix verification |
| Part 4 | Independent edge noise and photon-presence baseline |
| Part 5 | Open-boundary minimum-weight matching and syndrome consistency |
| Part 6 | Monte Carlo logical-failure estimation |
| Part 7 | Small finite-size pilot and statistical interpretation |
| Part 8 | Optional Stim/PyMatching repeated surface-code memory benchmark |
| **Part 9** | **Orientation-dependent fault probabilities and weighted MWPM comparison** |

## Installation and execution

Python 3.10 or later is recommended. In the project directory:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install numpy pandas networkx matplotlib ipython ipykernel notebook
jupyter notebook Topological_Detector_Graph_Decoding.ipynb
```

Select the Python environment as the kernel and choose **Run All**. The core experiment uses only the packages shown above. Plots and tables are displayed directly in the notebook; the repository does not require a separate `src/` directory or external input files.

The **optional** specialist example uses [Stim](https://github.com/quantumlib/Stim) and [PyMatching](https://pymatching.readthedocs.io/):

```bash
python -m pip install stim pymatching
```

If these packages are absent, the notebook skips that section and continues normally. The example uses a repeated rotated surface-code memory whose syndrome history has a three-dimensional spacetime structure; it is **not** a simulated photonic RHG preparation circuit.

## Reproducibility and checks

The complete notebook was executed from a fresh Python kernel, including both the base model and the extension. The core numerical execution does not depend on Stim or PyMatching. Checks include:

- incidence-matrix dimensions and one- or two-detector support per candidate fault;
- exhaustive single-edge syndrome-column consistency for a small lattice;
- correction syndromes matching the observed detector events;
- a boundary-to-boundary zero-syndrome chain with nontrivial logical parity;
- deterministic zero-error limits and valid Monte Carlo output ranges;
- equal graph and logical-observable definitions in the base model and extension;
- syndrome reconstruction by the nonuniform weighted decoder;
- exact shortest boundary-spanning lengths for $d=3,4,5$ and isolated-fault boundary ambiguity;
- increasing likelihood weights as fault probabilities decrease, and the zero-fault limit.

Stochastic examples use explicit random seeds. Verification of these algebraic and numerical properties does not establish a physically accurate noise model or logical-code threshold.

## Limitations

1. **Phenomenological rather than circuit-derived.** Detector-graph edges are assigned independent Bernoulli faults; no emitter, optical fusion, or cluster-state measurement circuit produces these probabilities.
2. **Simplified geometry and boundaries.** The cubic graph and its logical cut are defined for an illustrative matching problem, not proven equivalent to the primal/dual RHG lattice and its logical operators.
3. **No physical erasure decoding.** The all-photon-presence baseline does not implement erasure-aware fault tolerance.
4. **Limited scalability.** The NetworkX-based matching implementation recomputes many shortest paths and is intended for small systems; the optional specialized benchmark is a different code family. A Dijkstra-path precomputation could improve performance, but was not introduced here because it is unnecessary for the short pilot and could alter numerical tie-breaking.
5. **Insufficient statistics for threshold analysis.** The small pilot studies provide methodology checks, not threshold estimates, asymptotic scaling laws, or hardware performance forecasts.

A physical RHG study would require a specified cluster-state measurement circuit, a validated detector error model including realistic losses and correlated faults, an erasure-aware decoding strategy, and controlled comparisons over meaningful code distances.

## References

- R. Raussendorf, J. Harrington, and K. Goyal, *A fault-tolerant one-way quantum computer*, [arXiv:quant-ph/0510135](https://arxiv.org/abs/quant-ph/0510135).
- R. Raussendorf, J. Harrington, and K. Goyal, *Topological fault-tolerance in cluster state quantum computation*, *New Journal of Physics* **9**, 199 (2007), [DOI: 10.1088/1367-2630/9/6/199](https://doi.org/10.1088/1367-2630/9/6/199).
- S. Bartolucci *et al.*, *Fusion-based quantum computation*, *Nature Communications* **14**, 912 (2023), [DOI: 10.1038/s41467-023-36493-1](https://doi.org/10.1038/s41467-023-36493-1).
- [Stim documentation and source](https://github.com/quantumlib/Stim).
- [PyMatching documentation](https://pymatching.readthedocs.io/en/stable/).
