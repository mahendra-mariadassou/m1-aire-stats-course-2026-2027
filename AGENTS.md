# AGENTS.md

Statistics course website (AIRE Master M1, 2026-2027) by Mahendra Mariadassou. Mixed R + Python material, deployed as a GitLab Pages site.

## Structure / toolchain (they differ per folder — don't assume one)

- `README.md`: hand-written course index; this is the **site homepage source** (CI renders it to `index.html` via pandoc). Keep lecture/tutorial/homework links here.
- `Lectures/*.Rmd`: lecture slides, rendered with **xaringan** (`xaringan::moon_reader`, remark.js). Chakra is `libs/remark-latest.min.js` + `css/custom_*` **relative to the Rmd** (so `Lectures/libs/` is the needed dir; the root-level `libs/` is an empty unused placeholder). `macros.html` is included `before_body`. Knit with `cache = TRUE`.
  - Build: `rmarkdown::render("Lectures/01_Basics.Rmd")` (or Knit in RStudio). Depends on `tidyverse`.
- `Tutorials/*.qmd`: notebooks, **Quarto** `html:` format (except `04_linear_regression.qmd` which is `revealjs:` slides). They are **either Quarto/Python or Quarto/R** — some use `jupyter: python3` (e.g. `01_titanic_notebook.qmd`), some use the knitr/R engine (e.g. the R twin `01_titanic_notebook_r.qmd` with `engine: knitr`), whatever the kernel the file's chunks are written in. Both are built the same way: `quarto render Tutorials/<file>.qmd`. NB: there are two tutorials numbered `04` (`04_linear_regression.qmd` and `04_maximum_likelihood.qmd`).
- `Homework/HW*/aire*.qmd`: Quarto, either **Quarto/Python or Quarto/R** depending on the kernel used in the chunks. Build with `quarto render Homework/HW*/aire*.qmd`. Two paired files per HW: the assignment (`aire2026_homeworkN.qmd`) and the correction (`aire2026_homeworkN_correction.qmd`). Datasets live in a sibling `data/` dir. (E.g. `Homework/HW1_Penguins/` uses `{python}` chunks; in `Homework/HW2_IPS/` the assignment uses `{python}` but the correction uses `{r}` chunks — so always check each file's kernel language.)
- `Exercices/homework_mle_exercises.qmd`: Quarto/html, same author/`lightbox`/`code-fold` header conventions as tutorials/HW. Its `.html`+`_files/` **and** `.pdf` are all committed (the `.pdf` was force-added despite the `*.pdf` gitignore rule — new PDFs are still ignored).
- `Quiz/quizzN/`: R **exams** package. `prepare_quiz_N.R` renders `questions/question_<n><a|b|c>.rmd` variants, generates NOPS paper sheets (`exams2nops` → `pdftk` merge), and emails graded results via `gmailr`. `nchoice=4`, per-question variants are selected by letter suffix. `Quiz/prepare_quiz.R` (un-numbered) is the generic/main version (`nops_eval` → `gmailr`) reading the register CSVs.

## Git / build conventions (non-obvious)

- **Generated outputs are committed**: `.html` and `<file>_files/` (figures, libs) are tracked so the site works. Gitignored: `*_cache/` dirs, `*.pdf` (note `Exercices/homework_mle_exercises.pdf` exists on disk and IS tracked — force-added), `.Rproj.user`, `.Rhistory`, `.RData`, `.quarto/`, `**/*.quarto_ipynb`.
- `Homework/HW1_Penguins/`, `Homework/HW2_IPS/`, `Exercices/`, `Quiz/`, and all `Tutorials/*.qmd` (with their `<file>_files/`) are committed — for each HW both the assignment and the correction (`.qmd` + `.html` + `_files/`), plus their `data/*.tsv` datasets. Currently the **only untracked** file is the root `nops_eval.csv` (transient grading artifact). `moodlequiz.xml` (previously untracked) has been removed. **Never claim a committed file is uncommitted (or vice-versa) without `git ls-files`/`git status` first** — the tracked set changes often.
- `Quiz/.gitignore` policy (in root `.gitignore`): only root `*.csv`, `quizz*/questions/*.rmd`, and `quizz*/prepare_quiz*.R` are tracked under `Quiz/`; everything else there (scans, blank sheets, corrections, resultats, all_exams.pdf) is ignored.
- `Exam/` is an empty placeholder. Data files may ship as `.csv`, `.tsv`, or `.zip` (e.g. `Tutorials/data/bike_data_2019.zip`).

## Deploy

- `.gitlab-ci.yml`: GitLab Pages. On default branch only. Runs `pandoc --toc -s README.md --template template.html -o index.html`, then `cp -r * .public` — so any generated file reachable from README must exist in the repo (committed outputs, see above).

## Conventions to preserve

- Tutorial/HW qmd YAML header uses the same author block (Mariadassou, INRAE, ORCID `0000-0003-2986-354X`, email `mahendra.mariadassou@inrae.fr`) and `lightbox: true`; homeworks/tutorials also use `code-fold: show`.
- Quiz register files are **separator-mismatched by design**, don't "fix" them: `M1_AIRE_Students.csv` (nops_eval register) is **semicolon**-separated; `M1_AIRE_Students_Full.csv` (email join for grading) is **comma**-separated.
- Lecture Rmd setup: `knitr::opts_chunk$set(echo = FALSE, cache = TRUE)` and `theme_set(theme_classic()...)`.
- Titanic tutorials name matching chunks with **Quarto `#| label: name`** syntax (same label across the Python and R twins, e.g. `importLibraries`, `loadData`, `histogramAge`). R-only helper chunks (`showColumnNames`, `defineHelpers`) have no Python counterpart. Figure files follow the chunk label (e.g. `histogramAge-1.png`).
- Dataset URLs on the site point to the in-repo file: `.../m1-aire-stats-course-2026-2027/Tutorials/data/<file>` (don't use external `raw.githubusercontent.com` links, which are broken).
