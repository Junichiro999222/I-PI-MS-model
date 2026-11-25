# Impact-Induced Gas Photoionization Mass Spectrometry (I-PI-MS)

Python/Jupyter notebooks for modeling the downstream analytical response of an impact-induced gas photoionization mass spectrometer and investigating instrument design sensitivities for planetary dust analysis.

## Overview

The notebooks implement a three-dimensional model of neutral-gas density, vacuum-ultraviolet (VUV) photon transport (including photoabsorption and Rayleigh scattering), molecular photoionization, and ion collection. The sensitivity analysis examines gas pressure, inlet and lamp positions and orientations, relative VUV power, and mesh/repeller configuration for o-xylene, ethanol, and acetone.

The model uses prescribed molecular gas supply conditions. It does not independently predict impact-generated vapor production, molecular survival during hypervelocity impacts, or transient gas transfer from the impact region to the ionization chamber.

## Notebooks

| Notebook | Contents |
| --- | --- |
| `I_PI_MS_Modelling.ipynb` | Model geometry, optical inputs, neutral-gas distributions, ion collection, and modeled ion-generation and detection rates. Also contains laboratory QMS data-processing and model-comparison routines, which require separately supplied experimental data. |
| `I_PI_MS_Sensitivity_Analysis.ipynb` | Parameter sweeps, species-dependent sensitivity, geometry optimization, and relative electrical-power sensitivity. |

The model and sensitivity analysis can be examined independently of the experimental spectra. The experimental comparison is a separate, optional workflow requiring the original QMS input files.

## Installation

The development environment used Python 3.11.5. Install the listed dependencies in a dedicated environment:

```bash
python -m pip install -r requirements.txt
python -m notebook
```

Open Jupyter from the repository root. By default, the notebooks use the current working directory (`Path.cwd()`) to locate input files. Adjust `DATA_DIR` if using another directory.

## Optical input data

The optical CSV files are not distributed in this repository. The literature-based cross sections were digitized using WebPlotDigitizer from published sources. The VUV spectral input is based on manufacturer spectral information. The following table records the filenames expected by the notebooks and their source attribution.

| Required filename | Quantity | Source | Input x-axis |
| --- | --- | --- | --- |
| `0-xylene_PI_CS.csv` | o-Xylene photoionization cross section | Zhou et al. (2009) | eV |
| `acetone_PI_CS.csv` | Acetone photoionization cross section | See the associated manuscript and original digitization record | eV |
| `ethanol_PI_CS.csv` | Ethanol photoionization cross section | Person and Nicole (1971), as cited in the notebook; verify bibliographic spelling | eV |
| `xylene_abs.csv` | Benzene photoabsorption cross section, used as a surrogate for o-xylene | Rennie et al. (1998) | nm |
| `acetone_abs.csv` | Acetone photoabsorption cross section | Nobre et al. (2008) | eV |
| `ethanol_abs.csv` | Ethanol photoabsorption cross section | Feng and Brion (2002) | eV |
| `Oxyegen_VUV.csv` | Molecular oxygen photoabsorption cross section, used for background-gas absorption | Lu et al. (2010) | nm |
| `VUV.csv` | VUV lamp spectral distribution | Hamamatsu Photonics L10706 spectral information | nm |

To prepare local inputs:

1. Consult the original publications and manufacturer documentation referenced in the associated manuscript. Obtain the appropriate numerical data or digitize the relevant curves, subject to the applicable rights and access conditions.
2. Save each optical input as a headerless, comma-separated, two-column CSV: the first column is the energy or wavelength as listed above; the second column is the cross section in megabarns (Mb) or, for `VUV.csv`, the spectral ordinate in the source-compatible convention. Use 1 Mb = 10^-18 cm². Verify source units before conversion and check the lamp ordinate convention before applying a scale factor.
3. Retain the filenames exactly as listed, including their existing spelling. Place the files in `DATA_DIR`. The notebooks handle sorting, wavelength-to-energy conversion where required, smoothing, interpolation, and the model-specific corrections described in the manuscript and code. Do not apply these corrections twice.

The listed bibliographic attributions should be read together with the full reference list in the manuscript. Independently digitized curves can differ from the original model inputs; exact numerical agreement is not guaranteed.

## VUV calibration and numerical reproducibility

The absolute VUV spectral scaling factor (`VUV_SPECTRAL_SCALE`) used in the study was derived with reference to non-public manufacturer-provided information and is not distributed. The underlying non-public manufacturer data are also excluded.

To run either notebook, supply a locally determined, finite positive VUV spectral scaling factor compatible with the units and normalization of your own `VUV.csv` and the irradiance convention used in the notebook (µW cm⁻² eV⁻¹ at the reference plane, 50 cm from the lamp window). Set the environment variable `IPI_MS_VUV_SPECTRAL_SCALE` **before starting Jupyter**. For example, in Windows PowerShell:

```powershell
$env:IPI_MS_VUV_SPECTRAL_SCALE = Read-Host "Enter your own calibrated VUV scaling factor"
python -m notebook
```

On macOS/Linux:

```bash
read -r -p "Enter your own calibrated VUV scaling factor: " IPI_MS_VUV_SPECTRAL_SCALE
export IPI_MS_VUV_SPECTRAL_SCALE
python -m notebook
```

The settings cell rejects missing, non-numeric, non-finite, and non-positive factors. No study-specific value or demonstration default is supplied. Restart the kernel after changing the environment or calibration. Do not commit a locally modified notebook containing a private calibration value or saved outputs. A factor from a different input spectrum or calibration geometry is not necessarily transferable. Do not interpret an arbitrary demonstration factor as an experimentally calibrated irradiance.

The model structure and relative design sensitivities can be investigated using user-supplied optical inputs and a consistent local scaling factor. Absolute photon fluxes, ion-generation rates, detection sensitivities and other calibrated results depend on that factor and the optical input data. This public code release therefore does not by itself guarantee numerical reproduction of every reported value.

## Experimental QMS data

The experimental mass spectra acquired for this study are not included in this code repository. They are available from the corresponding author upon reasonable request, subject to applicable institutional and research-data policies. The experimental comparison routines require the four original instrument-export CSV files (CP932 encoding; instrument-specific layout) in their expected relative paths: `241230/1.csv` (o-xylene EI), `241230/4.csv` (o-xylene PI), `250129/6.csv` (mixture EI), and `250129/5.csv` (mixture PI). Set `RUN_QMS_ANALYSIS = True` in the modelling notebook only if all four files are available; the default is `False`. The QMS-dependent figures, tables, and INMS comparison are automatically skipped by default. They are not required for the standalone sensitivity-analysis notebook.

## Running the analysis

- Prepare the optical inputs and specify a local VUV calibration factor before running the calculations.
- Run the numerical-model sections of `I_PI_MS_Modelling.ipynb` in sequence. The experimental-spectra processing, experimental/model comparison, and experimental sensitivity calculations require the additional QMS files; these sections and their dependent calculations are automatically skipped unless explicitly enabled.
- Run `I_PI_MS_Sensitivity_Analysis.ipynb` sequentially using the optical inputs and the same local calibration convention. Output controls are defined in the settings cell; `SAVE_OUTPUTS = False` by default, including the final joint-optimization CSV exports. Set it to `True` to write results to `FIGURE_DIR`. The modelling notebook uses `SAVE_FIGURES = False` by default.
- Generated output files and locally supplied data are not part of the public source-code distribution.

## Associated publication and citation

This repository accompanies the manuscript on fundamental characterization of I-PI-MS for dust analysis in space exploration. Please cite the associated article and the archived version of the software when using this code. Publication and software archive identifiers will be added when available.

Please also cite the original literature sources when using the optical input data.

## License and third-party materials

A software license should be selected by the rights holder before public release. Any software license accompanying the repository applies only to the original software covered by it. Third-party data, manufacturer documentation, and other externally owned materials are not relicensed by this repository. Their omission does not imply a restriction on any independent rights users may have to obtain and use those materials.

## Contact

For questions about the model or requests for experimental data, please contact the corresponding author of the associated publication. Access to non-public third-party information is subject to the relevant rights holder's permission.
