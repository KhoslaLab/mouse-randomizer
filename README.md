# 🐭 Mouse Randomizer

A browser-based tool for balanced random group assignment in preclinical mouse
studies. Upload a CSV with animal IDs, sex, body weight, and bone mineral
density (BMD), and the app will assign mice to treatment groups while balancing
on sex, weight, and BMD.

**[Launch the app →](https://khoslalab.github.io/mouse-randomizer/)**

## Features

- **Balanced randomization** — Mice are stratified by sex and assigned using a 
  composite z-score of weight and BMD, so groups end up with similar 
  distributions across all three variables.
- **Flexible group configuration** — Choose 2–8 groups with custom labels 
  (defaults to "Vehicle" and "AP"). Groups can be added or removed as needed.
- **Incremental assignment** — Upload a CSV that already has some mice assigned 
  to groups (e.g., from a previous cohort) and leave the "Group" column blank 
  for new mice. The app will assign only the new mice while maintaining overall 
  balance with the existing assignments.
- **Per-group summaries** — After randomization, view group-level summaries 
  including sample size, sex breakdown, and average weight and BMD overall and 
  by sex.
- **CSV download** — Export the result as a CSV identical to your upload with an
  added "Group" column.
- **Template download** — Download a blank template CSV with the expected column
  headers.
- **Fully client-side** — No data leaves your browser. Everything runs in a 
  single HTML file with no server or external dependencies (aside from Google 
  Fonts).

## Usage

### 1. Prepare your CSV

Your CSV should include the following columns (naming is flexible):

| Column | Examples of recognized names |
|---|---|
| Animal ID | `Animal ID`, `Mouse ID`, `ID`, `Subject` |
| Sex | `Sex`, `Gender` |
| Weight | `Start Weight (g)`, `Weight`, `BW`, `Mass` |
| BMD | `Spine BMD`, `BMD`, `Bone Mineral Density`, `DEXA`, `DXA` |
| Group *(optional)* | `Group`, `Treatment`, `Assignment` |

If you don't have a CSV yet, click **Download template CSV** inside the app to
get a blank one with the correct headers.

### 2. Upload and configure

- Drag and drop your CSV onto the upload zone, or click to browse.
- Set the number of treatment groups and edit labels as needed.
- If your CSV includes a "Group" column with some mice already assigned, the app
  will detect them automatically and only assign the remaining mice.

### 3. Randomize and review

- Click **Randomize Groups** to run the assignment.
- Review the summary cards showing sample size, sex split, and average 
  weight/BMD (overall, male, and female) for each group.
- Click **Re-randomize** to generate a new assignment with a different random 
  seed.

### 4. Download

Click **Download CSV** to save the result. The output CSV matches your original
file with the "Group" column added or filled in.

## How the algorithm works

1. Weight and BMD are each standardized to z-scores across all mice (including 
   any pre-assigned ones).
2. A composite score is computed as the average of the two z-scores.
3. Mice are separated by sex, and each sex group is sorted by composite score.
4. Within each sex, mice are divided into blocks equal to the number of groups. 
   For each block, multiple random permutations of group assignments are tested,
   and the permutation that minimizes cumulative imbalance in composite score 
   across groups is selected.
5. Any remaining partial block at the end is assigned greedily, prioritizing sex
   balance and then composite score balance.
6. Pre-assigned mice are accounted for in the running group totals before any 
   new assignments are made.

## Deployment

This is a single `index.html` file. To host on GitHub Pages:

1. Push `index.html` to a repository.
2. Go to **Settings → Pages** and set the source to your main branch.
3. The app will be live at `https://yourusername.github.io/your-repo-name/`.

No build step, bundler, or server is required.

## AI disclosure

This software was developed with the assistance of Claude (Anthropic). The code,
design, and documentation were generated through an iterative conversation with
the AI model and reviewed by the project author. The randomization algorithm and
user interface logic were authored by Claude based on requirements provided by
the project author.

## License

This project is licensed under the [MIT License](LICENSE).
