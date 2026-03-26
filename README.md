# 🐭 Mouse Randomizer

A browser-based tool for reproducible covariate-adaptive randomization in
preclinical mouse studies. Upload a CSV with animal IDs, sex, body weight,
and bone mineral density (BMD), and the app will assign mice to treatment
groups while balancing on sex, weight, and BMD.

**[Launch the app →](https://khoslalab.github.io/mouse-randomizer/)**

## Features

- **Covariate-adaptive minimization** — The default algorithm is inspired
  by Pocock & Simon (1975). Each mouse is sequentially assigned to the
  group that minimizes marginal imbalance across sex, weight
  (tercile-binned), BMD (tercile-binned), and group size, with a
  biased-coin probability (p = 0.80) to preserve randomness. Each
  covariate is balanced independently so one cannot mask another.
- **Sex-specific covariate binning** — On by default. When enabled, weight
  and BMD tercile boundaries are computed separately for males and
  females. This ensures the bins capture within-sex variation (e.g. light
  vs. heavy males) rather than mostly recapitulating the sex difference
  itself. Can be toggled off for studies where male–female baselines are
  similar.
- **Stratified block randomization** — Also available as a simpler
  alternative when only sex balance is needed. Mice are stratified by sex
  and assigned in random permutation blocks.
- **Formal balance assessment** — After randomization, pairwise
  standardized mean differences (SMDs) are displayed for weight and BMD
  (overall, within-males, and within-females), along with the absolute
  difference in male proportion for sex balance. SMDs are color-coded
  against 0.1/0.2 thresholds with an overall verdict.
- **Within-sex summaries** — Group summary cards report mean ± SD for
  weight and BMD broken out by sex, since sex is a balancing variable and
  within-sex balance matters for downstream analysis.
- **Cage distribution check** — If your CSV includes a "Cage" column, the
  app flags any cages where all mice ended up in the same treatment group.
  If no Cage column is present, the app automatically infers cage numbers
  from the standard Animal ID format (`[text][cageNumber].[mouseID]`,
  e.g. `VQ10.1.1` → cage 10, `VQ16.L1` → cage 16), so cage balance is
  checked by default for data following this convention. Mouse identifiers
  after the dot can be numeric or alphanumeric.
- **Reproducibility and audit trail** — Optionally specify a seed for
  exact reproducibility. Download a full randomization report including:
  timestamp, algorithm parameters, PRNG name (Mulberry32), seed, input
  file SHA-256 hash, group summaries with within-sex statistics, the
  complete SMD table, cage warnings, individual assignments, and a
  suggested methods statement for your manuscript.
- **Allocation concealment** — Toggle blinded labels (A, B, C…) in the
  output CSV, with a separate downloadable key file for unblinding.
  Supports single-blind study workflows.
- **Incremental assignment** — Upload a CSV with some mice already
  assigned to groups and leave the "Group" column blank for new mice. The
  app assigns only the new mice while maintaining overall balance. Warns
  explicitly if pre-assigned labels don't match configured group labels.
- **Flexible group configuration** — Choose 2–8 groups with custom labels
  (defaults to "Vehicle" and "AP").
- **Unequal allocation ratios** — Assign different weights to each group
  (e.g., 2:1 or 3:1) to allocate more mice to a control or treatment arm.
  Both the minimization and stratified block algorithms respect the
  specified ratio while still balancing covariates. The ratio is documented
  in the randomization report and suggested methods statement.
- **Robust CSV parsing** — Uses [Papa Parse](https://www.papaparse.com/)
  v5.4.1 (inlined, no CDN dependency) for reliable handling of quoted
  fields, embedded commas, mixed line endings, and other CSV edge cases.
  Non-fatal parser diagnostics (e.g., unexpected delimiters, field count
  mismatches) are surfaced as warnings rather than silently ignored.
- **Strict input validation** — Sex values must be `Male`, `M`, `Female`,
  or `F` (case-insensitive); unrecognized values produce a clear error
  with the row number. Duplicate Animal IDs are rejected explicitly.
- **Fully client-side** — No data leaves your browser. Everything runs in
  a single HTML file with no server or backend dependencies.

## Usage

### 1. Prepare your CSV

Your CSV should include the following columns (naming is flexible):

| Column | Examples of recognized names |
|---|---|
| Animal ID | `Animal ID`, `Mouse ID`, `ID`, `Subject` |
| Sex | `Sex`, `Gender` — accepted values: `Male`, `M`, `Female`, `F` (case-insensitive) |
| Weight | `Start Weight (g)`, `Weight`, `BW`, `Mass` |
| BMD | `Spine BMD`, `BMD`, `Bone Mineral Density`, `DEXA`, `DXA` |
| Group *(optional)* | `Group`, `Treatment`, `Assignment` |
| Cage *(optional)* | `Cage`, `Cage ID`, `Housing` |

If no Cage column is present, the app will attempt to infer cage numbers
from the Animal ID column using the format
`[text][cageNumber].[mouseID]` (e.g., `VQ2.1` → cage 2, `VQ10.1.1` →
cage 10, `VQ16.L1` → cage 16). Mouse identifiers after the dot can be
numeric or alphanumeric.

If you don't have a CSV yet, click **Download template CSV** inside the
app to get a blank one with the correct headers.

### 2. Upload and configure

- Drag and drop your CSV onto the upload zone, or click to browse.
- Set the number of treatment groups and edit labels as needed.
- Adjust the **allocation ratio** if you need unequal group sizes (e.g.,
  3:1 to put three times as many mice in the control group). Leave all
  weights at 1 for equal allocation.
- Choose your randomization method: **Minimization (Pocock–Simon
  inspired)** for multi-covariate balancing, or **Stratified block
  randomization** for sex-only stratification.
- Optionally enter a seed for reproducibility, or leave blank for
  automatic seed generation.
- Toggle **Allocation concealment** if you need blinded group codes.
- **Sex-specific covariate binning** is on by default. It computes
  separate weight/BMD tercile boundaries per sex. Turn it off if male and
  female baselines are similar in your study.
- If your CSV includes a "Group" column with some mice already assigned,
  the app detects them automatically. You'll be warned if pre-assigned
  labels don't match your configured group labels.

### 3. Randomize and review

- Click **Randomize Groups** to run the assignment.
- Review the summary cards showing sample size, sex split, and mean ± SD
  for weight and BMD (overall, male, and female) per group.
- Check the **Balance Assessment** table for pairwise SMDs (overall and
  within-sex) and sex proportion differences.
- If a Cage column is present, review the **Cage Distribution Check** for
  potential confounds.
- Click **Re-randomize** to try a different seed.

### 4. Download

- **Download CSV** — The result CSV matches your original with the "Group"
  column added or filled in.
- **Randomization Report** — A plain-text report documenting all
  parameters, statistics, and a suggested methods statement. Keep this
  with your study records.
- **Blinding Key** — If allocation concealment is enabled, download the
  code-to-treatment mapping separately. Store securely until unblinding.

## How the algorithm works

### Minimization (default)

This implements a covariate-adaptive minimization procedure inspired by
the method described by Pocock and Simon (1975). Note that this
implementation has not been formally validated against their original
formulation.

1. Tercile boundaries are computed for weight and BMD across all mice
   (including any pre-assigned). When **sex-specific binning** is enabled
   (the default), boundaries are computed separately within males and
   females, and factor levels become sex-prefixed (e.g., M0/M1/M2 for male
   terciles, F0/F1/F2 for female terciles). This prevents pooled bins from
   merely recapitulating sex when baseline values differ between sexes.
2. Mice are processed in random order. For each mouse, the app computes
   the marginal imbalance that would result from assigning it to each
   candidate group, separately for each balancing factor:
    - Sex (categorical: M/F)
    - Weight tercile (low/mid/high)
    - BMD tercile (low/mid/high)
    - Total group size
3. For each factor, imbalance is measured as the range (max − min) of
   counts across groups for the relevant factor level. When unequal
   allocation ratios are specified, counts are normalized by each group's
   target proportion before computing the range, so the algorithm targets
   the specified ratio (e.g., 2:1) rather than equal group sizes.
4. The total imbalance for each candidate group is the sum of ranges
   across all factors.
5. The group(s) with the lowest total imbalance are preferred. A biased
   coin (p = 0.80) assigns to a preferred group; with probability 0.20, a
   group is chosen uniformly at random.
6. Pre-assigned mice are accounted for in the running factor counts before
   any new assignments.

### Stratified block randomization

1. Mice are separated by sex into two strata.
2. Within each stratum, mice are shuffled randomly, then divided into
   blocks. With equal allocation, block size equals the number of groups.
   With unequal allocation (e.g., weights 2:1), each block contains
   group slots proportional to the weights (e.g., [A, A, B]), so the
   target ratio is maintained within every complete block.
3. Each block receives a random permutation of group assignments.
4. Pre-assigned mice are respected; only unassigned mice are processed.

## Balance assessment

After randomization, the app reports:

- **Pairwise SMDs** for weight and BMD: the absolute difference in group
  means divided by the pooled within-group standard deviation. Reported
  overall and within each sex.
- **Absolute difference in male proportion** between each pair of groups
  (not an SMD, since sex is categorical).
- SMD thresholds: < 0.1 (green), < 0.2 (amber), ≥ 0.2 (red). These are
  conventional heuristics for assessing covariate balance, not formal
  significance tests. They should be interpreted in the context of your
  study design and sample size.

## Deployment

This is a single `index.html` file with Papa Parse inlined (no external
runtime dependencies aside from Google Fonts). To host on GitHub Pages:

1. Push `index.html` to a repository.
2. Go to **Settings → Pages** and set the source to your main branch.
3. The app will be live at `https://yourusername.github.io/your-repo-name/`.

No build step, bundler, or server is required.

## References

- Pocock SJ, Simon R. Sequential treatment assignment with balancing for
  prognostic factors in the controlled clinical trial. *Biometrics*.
  1975;31(1):103–115.

## AI disclosure

This software was developed with the assistance of Claude (Anthropic). The
code, design, and documentation were generated through an iterative
conversation with the AI model and reviewed by the project author. The
randomization algorithm and user interface logic were authored by Claude
based on requirements provided by the project author.

## License

This project is licensed under the [MIT License](LICENSE).
