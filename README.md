# Does the Brain's Representational Geometry Lose Its Tree-Like Structure Before a Seizure?

A small, math-first test extending a 2023 *Nature Neuroscience* finding into a disease-relevant question it did not originally ask.

## The finding this project extends

Zhang, Rich, Lee, Sharpee (2023), "Hippocampal spatial representations exhibit a hyperbolic geometry that expands with experience," *Nature Neuroscience* 26:131–139 (Salk Institute, Princeton, Janelia Research Campus/HHMI), showed that healthy hippocampal population activity during spatial navigation is organized according to hyperbolic, tree-like geometry rather than flat Euclidean geometry. The mechanism they identify: place field widths follow an exponential distribution, which mathematically induces this tree-like structure. A 2025/2026 follow-up from the same lab (Yao, Praturu, Sharpee) confirms this is "statistically hyperbolic in Gromov's sense," a classical, well-defined concept from metric geometry (Gromov, 1987), not something specific to place cells.

Their work studies healthy navigation only. It says nothing about what happens to this geometric organization during a pathological brain state.

## The question this project asks

Does the tree-like (low Gromov delta-hyperbolicity) organization of population brain activity break down, i.e., become measurably *less* tree-like, in the period before a seizure compared to normal interictal activity?

## The math: Gromov's delta-hyperbolicity

For any four points in a metric space, form the three possible pairings of pairwise distances, sort the three resulting sums, and take half the gap between the two largest. This is `delta(w,x,y,z)`. A perfectly tree-like space gives delta = 0 for every quadruple; the more a space deviates from tree structure, the larger delta grows. Checking all possible quadruples is computationally infeasible for any reasonable sample size, so delta-hyperbolicity is estimated here via random sampling of quadruples (a standard, documented practice in the network science literature for this exact reason), summarized as a 95th-percentile statistic robust to a small number of noisy points.

This was implemented from scratch, not via a library, and verified against two known analytic cases before being trusted on real data:
- A perfect tree metric (built from a balanced binary tree graph) must give delta = 0 exactly. It did.
- A set of points on a Euclidean circle must give a clearly nonzero delta, since a circle is not tree-like. It did (0.4252).

## Method

EEG population-activity vectors were extracted from each provided segment (band power per channel across short sub-windows, substituting "moment in time" for "location," the closest available analogue to place-cell population vectors in EEG data). Pairwise Euclidean distances between these state vectors were computed, and delta-hyperbolicity was estimated per segment.

**A correction applied during analysis**: the AES challenge's file convention groups preictal segments into blocks of six 10-minute files per continuous hour preceding a single seizure, recorded in each file's own `sequence` field (1-6). Six segments from the same seizure's lead-up hour are highly temporally correlated, not independent observations. The final analysis aggregates all population-state vectors within a block before computing delta-hyperbolicity, giving one value per real seizure event rather than per file. The correctness of this grouping was validated directly against each file's own `sequence` metadata (0 mismatches across all 18 preictal files) before being trusted.

**Data**: Kaggle "American Epilepsy Society Seizure Prediction Challenge" (2014), Patient_1 (human intracranial EEG, not the dog subjects also included in this dataset, chosen specifically to preserve the connection to human epilepsy).


## Results

**Initial analysis (treating each preictal file as independent, n=18):**

| | N segments | Mean delta | Std delta |
|---|---|---|---|
| Preictal | 18 | 0.6066 | 0.4004 |
| Interictal | 50 | 0.8724 | 0.0733 |

Mann-Whitney U test: p = 0.0741.

**Corrected analysis (aggregated to one value per real seizure block, n=3):**

Validation check confirmed the block-grouping assumption exactly against each file's own `sequence` field (0 mismatches across 18 files), giving high confidence the correction below is structurally correct, not a new source of error.

| | N segments | Mean delta | Std delta |
|---|---|---|---|
| Preictal | 3 | 0.6499 | 0.3605 |
| Interictal | 50 | 0.8724 | 0.0733 |

Mann-Whitney U test: p = 0.8421.

## Possible insights from the two results

 The initial n=18 analysis suggested a near-significant trend (p=0.074), but this treated six highly correlated files from the same continuous preictal hour as six independent observations. Once corrected to reflect the true number of independent seizure events for this patient (n=3, verified against each file's own metadata), the apparent trend collapses entirely (p=0.842). With only three independent preictal events, this test has essentially no statistical power to detect a real effect even if one exists, this is a genuinely underpowered null result, not evidence against the hypothesis.

 It is a direct, quantified demonstration of why treating correlated segments as independent observations matters: a suggestive-looking result at n=18 disappeared once the true independence structure was respected. This mirrors, in a controlled, single-project way, the same lesson investigated in a companion project of mine in this github repo ml-tinkering (patient-level data leakage in an Alzheimer's MRI staging model), non-independence inflates apparent evidence, and correcting for it is not optional methodological housekeeping, it changes conclusions.

 Preictal delta values showed higher variance than interictal values in both the flawed and corrected analyses (std 0.36-0.40 vs 0.07). With only three preictal data points, this is far too small a sample to draw a real conclusion from, but it may be worth testing directly, with adequate statistical power, in future work rather than dismissed outright.

 This patient had only three recorded seizures. A properly powered version of this test would need to pool data across multiple patients, using a method that respects the per-patient, per-seizure block structure (e.g., a mixed-effects or hierarchical approach, not naive pooling)

## Reliability of methods

Delta-hyperbolicity is not a learned, opaque confidence score. It is a direct, geometric computation with a precise mathematical meaning, "how far is this set of brain states from being organized as a tree", verifiable and interpretable by construction. Any result can be explained concretely by pointing to the specific quadruple of brain states that produced a given delta value.

## Limitations

- Single-patient scope, and after correcting for the true independence structure, only 3 independent seizure events. This project cannot and does not claim to have adequately tested the underlying hypothesis; it demonstrates that the hypothesis can be tested rigorously, and shows precisely what happens to the apparent result when independence is (and isn't) correctly accounted for.
- No cross-patient generalization claim is made (consistent with how the original AES challenge and much of this literature treats intracranial EEG, given electrode placement heterogeneity across patients).
- The EEG-to-population-vector substitution (time-window states in place of spatial-location states) is a reasonable but unverified analogue of the original paper's place-cell methodology; it has not been validated against ground-truth place-field data.
- This is a proof of concept demonstrating the method can be built, verified, and run honestly on real clinical data, including catching and correcting a real statistical error along the way, not a validated clinical tool or a settled answer to the underlying scientific question.


## References

- Zhang, Rich, Lee, Sharpee (2023), *Nature Neuroscience* 26:131–139.
- Yao, Praturu, Sharpee (2025/2026), bioRxiv.
- Gromov (1987), "Hyperbolic Groups," in *Essays in Group Theory*.
- American Epilepsy Society Seizure Prediction Challenge (2014), Kaggle.



