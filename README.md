# All-Back-Contact Perovskite/Silicon Tandem Solar Cell Model

Wolfram Language code for modeling quasi-interdigitated back-contact perovskite solar cell (QIBC PSC) / interdigitated back-contact (IBC) silicon tandems under location-dependent solar spectra and temperature conditions.

## Study overview

This study examines how electrical coupling and device losses influence the performance of all-back-contact perovskite/silicon tandems. Three configurations are compared:

| Configuration | Electrical connection | Operating principle |
|---|---|---|
| **2T** | Series-connected two-terminal tandem | The subcells carry the same current, and their voltages add. |
| **VMM** | Voltage-matched two-terminal module tandem | Perovskite and silicon strings operate at a common terminal voltage, and their currents add. |
| **4T** | Four-terminal tandem | The subcells operate at independent maximum-power points, and their powers add. |

The outdoor analysis uses fixed-tilt spectral irradiance data obtained from the National Solar Radiation Database (NSRDB) for **2020**, at **60-minute resolution**, for four locations:

- **Singapore** — tropical environment.
- **Phoenix, Arizona, USA** — hot, dry environment.
- **Los Angeles, California, USA** — temperate coastal environment.
- **Chicago, Illinois, USA** — continental environment.

The wider study also compares performance under standard AM1.5G illumination. The code documented here implements the time-resolved outdoor workflow; its default input paths select Singapore. The other locations require their corresponding inputs and run settings.

The reported efficiencies and energy yields are model predictions. The perovskite input characteristics include an optimized TCAD representation of the reference QIBC device and should be distinguished from measured device data.

## Model features

- Conversion of wavelength-resolved irradiance into photon-energy space.
- EQE-based photogeneration and an approximate optical-coupling model between the subcells.
- Temperature-dependent bandgaps, intrinsic carrier concentrations, and recombination terms.
- Radiative and nonradiative recombination, silicon Auger recombination, series resistance, and shunt leakage.
- Current-matched, voltage-matched, and independently operated tandem calculations.
- Optional parameter sweeps for the top-cell optical scaling factor and VMM voltage-matching ratio using a subset of daylight spectra.
- Time-resolved PCE, irradiance-weighted average PCE, and annual energy yield.
- Parallel processing, checkpoint recovery, CSV export, and PCE-versus-irradiance plots.

## Software requirements

- **Wolfram Mathematica**, including its notebook front end and kernel. The study notebook was prepared using version **14.3 on Windows**; compatibility with other versions has not been established here.
- Parallel kernels for the annual parallel workflow, subject to the local Mathematica installation and license.
- The device-response and environmental input files described below.

The code uses `NotebookDirectory[]`. Run it from a saved Mathematica notebook. If the downloaded source opens as plain text, place the code in Input cells in a new notebook and save it as a Wolfram Notebook (`.nb`) before evaluation. A standalone `.wl` script would require an explicit base-directory setup.

## Input files and folder layout

Arrange the local working directory as follows. This is the layout required by the import statements, rather than a claim that all inputs are already included in the repository.

```text
All-back-contact-tandems-realistic-model/
├── All-Back-Contact Tandems Model.nb
├── README.md
├── LICENSE
├── .gitignore
├── Input/
│   ├── JV-HTIBC-Si.csv
│   ├── EQE-HTIBC-Si.csv
│   ├── JV-QIBC-PSC.csv
│   ├── EQE-QIBC-PSC.csv
│   ├── Singapore-Spectra.xlsx
│   └── Singapore-Temperature.xlsx
└── Output/
```

| Input filename | Description |
|---|---|
| `JV-HTIBC-Si.csv` | Reference silicon current-density–voltage data |
| `EQE-HTIBC-Si.csv` | Reference silicon external quantum efficiency |
| `JV-QIBC-PSC.csv` | Optimized perovskite current-density–voltage data |
| `EQE-QIBC-PSC.csv` | Optimized perovskite external quantum efficiency |
| `Singapore-Spectra.xlsx` | Time-resolved Singapore spectral irradiance |
| `Singapore-Temperature.xlsx` | Corresponding Singapore temperature series |

Some device files have been uploaded under longer descriptive names, including `JV-Referenced-HTIBC-Si.csv`, `EQE-Referenced-HTIBC-Si.csv`, and `JV-Optimized-Referenced-QIBC-PSC.csv`. Either rename the appropriate local copies to the names expected above or update the import statements to match them. Verify the identity of the perovskite EQE dataset before assigning it to `EQE-ideal-1.62.csv`.

### Data format

- **JV CSV files:** numeric voltage–current-density pairs. Record the current-density units, area normalization, and sign convention with each dataset.
- **EQE CSV files:** numeric photon-energy–EQE pairs, with energy in **eV** and EQE as a **fraction**, not percent. The code interpolates these values directly.
- **Spectral workbook:** first worksheet; wavelength in **nm** across the first row from column B onward, timestamps in column A below the header, and spectral irradiance in **W m⁻² nm⁻¹** in the remaining cells. Convert source units where necessary.
- **Temperature workbook:** first worksheet; a header row followed by timestamps in column A and temperature in **kelvin** in column B. Convert Celsius values using `T[K] = T[°C] + 273.15`.
- Spectral and temperature timestamps must have the same ordering, time zone, and sampling interval. A complete hourly record for 2020 contains **8,784 intervals**.

The code applies the imported temperature directly to the device parameters. An ambient-temperature input therefore represents an ambient-temperature approximation; it does not become cell operating temperature through an implicit thermal model.

## Running a simulation

1. Download the repository and save the model as a Mathematica notebook.
2. Supply the required files in `Input/` and create `Output/` beside it.
3. Open the notebook and start a fresh kernel.
4. Set `spectraFile` and `temperatureFile` for the desired location. Check the imported units and timestamp alignment before continuing.
5. Select the architectures using `computeSeries`, `computeModule`, and `computeFour`.
6. Choose the optimization mode. With `autoOptimizeParameters = True`, Section 5.1 selects `thick` and `moduleRatio` from parameter sweeps. With `False`, it uses the specified values and rebuilds the thickness-dependent optical functions.
7. Evaluate the computational sections in order and inspect the Section 5.2 test results. Include illuminated intervals when assessing runtime and numerical behavior.
8. Run Section 6 for the full time series, then the export and plotting sections.
9. Preserve the inputs, parameter settings, outputs, and code version together for each run.

Use a separate working copy or checkpoint path for every location and parameter set. Do not resume a checkpoint generated with different inputs or model settings.

### Settings that affect interpretation

- `thick` is a dimensionless optical-response scaling parameter, not a thickness in nanometers.
- `moduleRatio` represents the silicon-to-perovskite series-count ratio used for voltage matching. A noninteger value is a model ratio and requires an appropriate physical string layout before fabrication.
- The parameter sweep selects values before the annual evaluation. It does not imply that a fabricated device changes its geometry every hour.
- `dv` controls the voltage-grid spacing. Check convergence before reporting final efficiencies.
- The current source uses `lowLightCutoff = 20.` W m⁻² and a numerical PCE screening threshold. These are numerical settings, not universal physical limits.
- The annual aggregation assumes uniform time spacing. Its current `safe` function replaces nonnumeric PCE entries with zero; inspect failed or rejected intervals before interpreting annual totals.

## Outputs

| Output | Contents |
|---|---|
| `AnnualTandemResults.csv` | Timestamp, irradiance, 2T/VMM/4T PCE, and temperature |
| `Tandem_Scatter_CommonIrradiance.csv` | PCE-versus-irradiance scatter data |
| `Tandem_MovingAverage_CommonIrradiance.csv` | Moving-average data |
| `Tandem_Scatter_and_MovingAverage_SharedX.csv` | Combined plotting table |
| `PCE_vs_Irradiance_MovingAverage.png` | PCE-versus-irradiance figure |

These files are written to `Output/`. The notebook also prints annual energy yield in **kWh m⁻² year⁻¹** and irradiance-weighted average PCE in **%**. Moving averages are plotting summaries; annual metrics are calculated from the time-resolved results.

To reproduce a manuscript case, use its exact device parameters, loss settings, input data, and optimization configuration. The default configuration alone should not be assumed to reproduce every figure or table.

## Original model and modifications

This code is a modified derivative of **RealisticTandem.nb** from **Moritz H. Futscher and Bruno Ehrler**.
[https://doi.org/10.1021/acsenergylett.7b00596](https://doi.org/10.1021/acsenergylett.7b00596).

The present adaptation, by **Waqas Ahmad (2026)**, applies the original tandem-modeling framework to the annual performance analysis of QIBC perovskite/IBC silicon tandems in series-connected 2T, voltage-matched module (VMM), and four-terminal (4T) configurations. Modifications include device-specific JV and EQE inputs, material parameters and loss settings, and the processing of hourly spectral irradiance–temperature pairs for Singapore, Phoenix, Los Angeles, and Chicago during 2020. The adapted implementation also includes parameter sweeps for the perovskite optical-response scaling factor and VMM voltage-matching ratio, temperature-dependent recombination calculations, parallel annual evaluation with checkpoint recovery, and automated export of hourly PCE, irradiance-weighted annual efficiency, annual energy yield, and plotting data.

## License

The adapted code is distributed under the **GNU General Public License, version 3 or any later version (GPL-3.0-or-later)**. See [LICENSE](LICENSE). Original copyright, license, and warranty notices must be retained, and redistributed modifications must comply with the applicable GPL terms.

Third-party datasets retain their applicable terms; the software license does not automatically relicense those datasets.

## Citation

When using this adaptation, please cite the original Futscher–Ehrler paper and identify the version or commit of this repository:

[https://github.com/wsahmad95-hub/All-back-contact-tandems-realistic-model](https://github.com/wsahmad95-hub/All-back-contact-tandems-realistic-model)

The associated manuscript is titled **“Electrical Coupling Governs the Outdoor Performance of All-Back-Contact Perovskite/Silicon Tandem Solar Cells.”** Its publication citation and a version-specific software DOI can be added when available.

## Questions and reproducibility reports

Please use the repository's [Issues page](https://github.com/wsahmad95-hub/All-back-contact-tandems-realistic-model/issues) for questions or reports. Include the code version, Mathematica version, relevant settings, and the smallest example that demonstrates the issue.
