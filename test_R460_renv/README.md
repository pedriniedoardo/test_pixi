# test_R460_renv

R 4.6 environment: pixi provides R and the system libraries, renv manages the R packages.

## Usage

```bash
pixi install        # create/update the env from pixi.toml + pixi.lock
pixi shell          # activate the env
rstudio .           # RStudio inherits PATH, so it uses the pixi R
```

```r
renv::restore()     # install the R packages recorded in renv.lock
renv::snapshot()    # record new packages in renv.lock
```

RStudio picks the first `R` on `PATH` (or `RSTUDIO_WHICH_R` if set), so launch it from `pixi shell` or `pixi run rstudio .` to get the pixi R and not the system one.

## Reusing this setup in a new project

Copy:

- `pixi.toml`, `pixi.lock` (adjust `name` in `pixi.toml`)
- `.R/Makevars`
- `renv.lock`
- `renv/activate.R`, `renv/settings.json`
- `.Rprofile`

Do not copy:

- `renv/library/`: compiled packages embed absolute paths to the old `.pixi` env
- `renv/staging/`, `renv/sandbox/`
- `.pixi/`, `.Rproj.user/`

Then in the new folder:

1. `pixi install`
2. `pixi shell`, then `rstudio .` (`renv/activate.R` bootstraps renv)
3. `renv::restore()`

Packages already in the renv global cache (same R version and platform) are linked instead of rebuilt.

## Notes

- **Do not add `r-renv` to `pixi.toml`.** conda-forge has no `r-renv` build for R 4.6 yet, so the solve fails. renv installs itself from CRAN via `.Rprofile`.
- **`systemfonts` fails with `ft2build.h: No such file or directory`.** `fontconfig.pc` requires `expat`, and only `libexpat` (runtime) was installed, so `pkg-config --cflags` failed and returned no include paths. Fix: `pixi add expat fontconfig freetype`, restart R, retry. Check with `pkg-config --cflags fontconfig freetype2`, which should print `-I.../include/freetype2`.
- **Other conda channels** (e.g. bioconda): add to `channels` in `pixi.toml`; the first channel has priority. CRAN/Bioconductor mirrors are set on the R side (`.Rprofile` or `renv.lock`), not in pixi.
- `pixi add <pkg>` only edits the named entries and keeps locked versions where it can. Check with `git diff pixi.toml pixi.lock`.
