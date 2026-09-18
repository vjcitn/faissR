# Bioconductor-oriented review of `faissR`

## Scope of review

This review focuses on Bioconductor expectations around dependency provenance, use of non-standard repositories, and access to external data sources, and it also comments on the scientific utility of the package.

## Overall assessment

`faissR` is scientifically relevant to Bioconductor because it provides nearest-neighbour search, k-nearest-neighbour prediction, and k-means tooling for dense low-dimensional representations that are common in single-cell, flow cytometry, imaging, and related omics workflows. The package is especially useful where users need faster exact or approximate neighbour search on PCA-like embeddings and where GPU acceleration may be valuable.

From a Bioconductor standards perspective, the package is close to expectations in several important areas:

- `DESCRIPTION` does **not** declare `Remotes` or `Additional_repositories`.
- R-level dependencies are standard CRAN/Bioconductor packages.
- The installed example data are bundled in `inst/extdata` and used offline by the vignette.
- The package appears to avoid runtime acquisition of remote data in the exported R interface.
- GPU reliance is marked optional in `.BBSoptions`.

However, there are still review points that should be addressed or watched carefully before treating the package as fully aligned with Bioconductor norms.

## Findings

### 1. Package metadata is mostly compatible with Bioconductor expectations

The `DESCRIPTION` file uses standard package metadata and Bioconductor-facing fields, including `biocViews`, and it limits package dependencies to CRAN/Bioconductor packages plus an external system requirement on FAISS. I did not find `Remotes` or `Additional_repositories`, which is a positive sign for Bioconductor review.

### 2. The package itself still promotes GitHub installation in user-facing documentation

A main concern is that the top-level `README.md` recommends:

- installing `remotes`
- installing `faissR` with `remotes::install_github("tkcaccia/faissR")`

This is not ideal for Bioconductor-facing documentation. For Bioconductor packages, the preferred installation path should be `BiocManager::install()` once the package is available there, and GitHub installation should not be the primary documented route. Keeping GitHub installation as the leading instruction increases reliance on a non-standard repository and weakens the Bioconductor presentation.

### 3. External native software provenance is explicit, but still a deployment risk

The package requires the FAISS C++ library as an external system dependency. That dependency is clearly declared in `SystemRequirements`, and the documentation is explicit that FAISS is not vendored or silently installed by the package itself. This is good practice.

The remaining risk is practical rather than hidden: package functionality depends on a third-party native library that is not part of standard R/Bioconductor repositories. Bioconductor can accept external system dependencies, but long-term build reproducibility depends on stable provisioning of FAISS on supported builders. The repository's `.prepare` logic and installation documents show active effort to manage this, but the dependency remains a review-sensitive point.

### 4. Documentation includes optional installation paths that rely on non-standard ecosystems

The installation materials mention several non-standard acquisition paths for native dependencies, including:

- GitHub source checkout of FAISS
- Homebrew
- conda/mamba

These may be reasonable for end users, but they are not Bioconductor-standard sources. Their presence is not automatically disqualifying, because they are documented as external system setup rather than R package dependencies. Still, Bioconductor-facing documentation should make clear which route is the preferred supported path for Bioconductor builds and which routes are only optional user/developer conveniences.

Particularly noteworthy is the documented macOS path where `FAISSR_AUTO_INSTALL_FAISS=1` can allow a GitHub installation flow to call Homebrew. Although this is opt-in rather than silent, it is more aggressive than typical Bioconductor practice and may attract reviewer scrutiny.

### 5. Remote data access in the installed package appears well controlled

I did not find evidence that exported package functionality downloads data at runtime. The example workflow uses a bundled RDS file from `inst/extdata`, and the provenance for that artifact is documented.

The script `inst/scripts/prepare_zeisel_brain_pca.R` does perform an upstream data acquisition step, but it is a preparation script rather than installed runtime behavior. Importantly:

- it uses `scRNAseq::ZeiselBrainData()`, i.e. an upstream Bioconductor data source
- the installed vignette uses the prebuilt offline artifact instead of redownloading data
- the package records provenance metadata for the bundled object

This is aligned with good Bioconductor practice. It avoids dependence on ephemeral direct-download URLs in routine package use.

### 6. Example data provenance is a strength

The bundled `zeisel_brain_pca.rds` object is compact, documented, and tied to an established public Bioconductor resource (`scRNAseq`) and the primary publication. The preparation script records source package, DOI, access date, preprocessing steps, and package versions. That is a strong reproducibility feature.

### 7. Scientific utility is strong and appropriate for Bioconductor workflows

The package has clear scientific utility for analysis pipelines built around dense embeddings such as PCA coordinates from single-cell and related assays. Likely areas of value include:

- nearest-neighbour graph construction and validation
- approximate neighbour search for large cell atlases
- kNN classification or label transfer on reduced dimensions
- k-means clustering on dense low-dimensional representations
- benchmarking CPU versus GPU neighbour-search backends in high-throughput settings

The utility is strongest for advanced users working at scale, especially where FAISS infrastructure is already available. The main limitation is accessibility: the external FAISS dependency raises the installation barrier relative to pure-R or header-only alternatives.

## Conclusions

`faissR` has strong scientific relevance for Bioconductor and shows good practice in avoiding hidden runtime downloads and in documenting bundled data provenance. The most important Bioconductor-facing concern is not remote data acquisition but reliance on non-standard software distribution channels in the installation story, especially the prominence of GitHub installation for the package itself and optional Homebrew/conda/GitHub-native dependency paths.

## Recommendations

1. Make `BiocManager::install("faissR")` the primary installation instruction once the package is in Bioconductor.
2. Demote `remotes::install_github()` to a development-only or bleeding-edge option.
3. Keep emphasizing that runtime package use is offline and does not fetch remote data.
4. Keep the bundled example data path and provenance model; this is a positive aspect of the package.
5. Continue documenting FAISS as an explicit external system dependency and separate supported Bioconductor build paths from optional user convenience paths.

## Bottom line

I would characterize the package as **scientifically useful and reasonably careful about data provenance**, with the **main Bioconductor compliance risk concentrated in installation/dependency provenance rather than in runtime data access**.
