# Warning on budget_sweep_full_space.csv

This curve is computed on the full 456-attribute point-in-time space, which includes
the 416 attributes that are constant within one ecosystem. Its early saturation
reflects the selection of ecosystem indicators rather than of informative metadata:
at k = 2 the curve already reaches AU-PR 0.966, because two ecosystem indicators
suffice to separate a dataset in which NPM is 83.5% malicious and PyPI 9.6%.

The compressibility result reported in the manuscript uses
`figures/figure04_budget_sweep_data.csv`, computed after ecosystem-identifying
attributes are removed. On that curve, five features attain AU-PR 0.9864 against
0.9912 at eighteen and 0.9910 at forty, all within one bootstrap interval width.

This file is provided for completeness and to document the ecosystem confound
discussed in Section 3.5 of the manuscript. It should not be read as a
compressibility result.