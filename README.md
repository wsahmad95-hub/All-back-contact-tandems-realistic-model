# All-Back-Contact Tandems Realistic Model

Wolfram Language model of quasi-interdigitated back-contact (QIBC) perovskite / interdigitated back-contact (IBC) silicon tandem solar cells under time-resolved spectral irradiance and temperature conditions.

The framework compares three electrical configurations:

- **2T:** current-matched, series-connected two-terminal tandem.
- **VMM:** voltage-matched module tandem with parallel-connected subcell strings.
- **4T:** four-terminal tandem with independently operated subcells.

The notebook includes spectral conversion, temperature-dependent device parameters, recombination and resistive losses, optical-response calculations, numerical optimization, and annual performance analysis. These are numerical model predictions, not measurements of fabricated tandem devices.

## Status and scope

The supplied notebook is `All-Back-Contact Tandems Model.nb`. Its current input paths select Singapore. Applying it to Phoenix, Los Angeles, or Chicago requires the corresponding spectral and temperature files and documented location-specific settings; the present paths do not run all four locations automatically.

This documentation was prepared from static inspection. The notebook has not been executed or independently validated as part of preparing this repository. Resolve the items in `PRE_RELEASE_CHECKLIST.md` before designating a manuscript release.

## Software

- Wolfram Mathematica with notebook front end and a licensed Wolfram kernel.
- The notebook metadata identifies Wolfram **14.3 for Microsoft Windows, 64-bit**. This is the authoring version, not a verified minimum supported version.
- The annual workflow uses parallel kernels (`LaunchKernels` and `ParallelMap`); availability depends on the local installation and license.

## Required input files

The notebook uses paths relative to `NotebookDirectory[]`. Place `Input` beside the notebook and supply the exact files below. These input datasets are not included in this documentation package.

| File inside `Input/` | Purpose |
|---|---|
| `IV-HTIBC-Si.csv` | Silicon current-voltage fitting data |
| `EQE-HTIBC-Si.csv` | Silicon external quantum efficiency data |
| `JV-Ideal-1.62.csv` | Perovskite current-voltage fitting data |
| `EQE-ideal-1.62.csv` | Perovskite external quantum efficiency data |
| `Singapore.xlsx` | Time-resolved spectral irradiance |
| `Temperature-Singapore.xlsx` | Corresponding temperature series |

### Spreadsheet layout read by the notebook

- **Spectra:** worksheet 1; first row contains wavelength values in nm from column B onward; subsequent rows contain a timestamp in column A and spectral values in the remaining columns.
- **Temperature:** worksheet 1; a header row followed by timestamps in column A and temperature in kelvin in column B.
- Both time series must have matching timestamps and row order. Document the time zone and time convention.
- The annual energy calculation assumes uniform time spacing. Verify the timestamps and document excluded or missing intervals. A complete hourly 2020 series contains 8,784 records.
- Spectral values must use the units expected by the wavelength-to-photon-flux conversion. Record the actual units and preprocessing in an input-data dictionary before release.
- The EQE files are interpolated directly and subsequently evaluated at photon energy. Verify the independent-variable units, EQE scale (fraction versus percent), and handling outside the measured range. Do not substitute wavelength-based files without the necessary conversion.
- Document the voltage/current units, normalization area, and sign convention of each JV file.

## Intended workflow

1. Complete the pre-release checks and supply the input files.
2. Save the notebook locally with `Input/` and `Output/` folders alongside it. Create `Output/` if it does not exist.
3. Open the notebook in Mathematica and start a fresh kernel.
4. Set the desired location's spectral and temperature paths. Use a separate output/checkpoint location for each dataset and parameter set.
5. Initialize the reference spectrum used by the early fitting cells (`AM1p5global`) before those cells are evaluated; the supplied notebook needs its reference-spectrum initialization documented or added.
6. Evaluate the computational sections in order, checking imports, timestamp alignment, numerical messages, and optimization convergence. Descriptive heading cells should not be executed as code.
7. Inspect the exported hourly results, verify a small independently checked example, and then complete the annual calculation.

Do not reuse a checkpoint after changing the location, inputs, model parameters, or code. Archive old checkpoints separately if they need to be retained.

## Outputs

The notebook contains exports for:

- `Output/AnnualTandemResults.csv`
- `Output/Tandem_Scatter_CommonIrradiance.csv`
- `Output/Tandem_MovingAverage_CommonIrradiance.csv`
- `Output/Tandem_Scatter_and_MovingAverage_SharedX.csv`
- `Output/PCE_vs_Irradiance_MovingAverage.png`

The annual CSV header contains time, irradiance (`PowerofSun`, W/m²), series-tandem PCE (%), module-tandem PCE (%), four-terminal PCE (%), and temperature (K). The notebook also includes annual energy-yield and irradiance-weighted efficiency calculations.

## Reproducing manuscript results

For a manuscript release, record the exact input files, parameter values, optimization bounds, temperature interpretation, and configuration for every reported case. State which parameters are fixed for the year and which are optimized during evaluation. Supply a table mapping each manuscript figure/table to the relevant run and output file.

Report the normalization area for VMM strings and the interpretation of the voltage-matching ratio. Do not interpret an optimized noninteger ratio as a directly fabricated fractional cell count.

## Attribution and reuse

Document the origin of every input dataset and distinguish experimental, digitized, and simulated data. Retain attribution and license notices for any reused code. The associated study identifies a methodological connection to DOI [10.1021/acsenergylett.7b00596](https://doi.org/10.1021/acsenergylett.7b00596); describe the actual code reuse and modifications before release.

A software license has not been selected in this preparation package. Add the appropriate `LICENSE` after confirming the applicable ownership and upstream terms. Third-party datasets may have separate reuse conditions.

## Citation

Before release, complete `CITATION.cff.template`, verify the software-author list, rename it `CITATION.cff`, and add the repository URL. Add a version-specific archival DOI when available. Add the associated manuscript citation once its publication details are available; do not describe it as accepted or published prematurely.

