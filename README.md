# trackver

Compares ECMWF ensemble tropical cyclone forecasts with what was observed.

For a storm you name, trackver downloads the latest [IBTrACS](https://www.ncei.noaa.gov/products/international-best-track-archive)
best-track observations and lists the ECMWF ensemble runs that started while the storm was
observed. You pick one, and it makes two plots of that run.

## Two modes

The repository has two programs. They make the same plots and take the same options, and differ
in where the forecasts come from:

| | `./trackver` | `./trackver_mars` |
| --- | --- | --- |
| forecasts from | the public S3 bucket `ecmwf-static-datasets` | MARS |
| models | IFS ensemble | IFS ensemble, plus the AIFS ensemble for runs from 2026-05-12 |
| needs | internet access only | the `mars` command (ECMWF's Atos HPC, or a workstation with MARS access) |
| example | `./trackver GARANCE` | `./trackver_mars DUJUAN` |

## The plots

**Meteogram:** the ensemble's maximum 10 m wind and central pressure at every 6-hour step, with
the observed value for each step on top.

![GARANCE meteogram, ECMWF ensemble from 2025-02-25 12 UTC against IBTrACS](docs/example_meteogram_garance.png)

**Plume map:** every member's track, coloured by lead time in 24-hour bands, with the
unperturbed members in black. The red dots are the observed storm positions at the forecast steps.

![GARANCE track plumes, ECMWF ensemble from 2025-02-25 12 UTC against IBTrACS](docs/example_plumes_garance.png)


## Installation

```
git clone https://github.com/alinalerneral/trackver.git
cd trackver
conda env create -f environment.yml      # or mamba / micromamba
conda activate trackver
```

Without conda, `pip install -r requirements.txt` works too, as long as `wget` is installed.
For `./trackver_mars`, `mars` must also be on the PATH; on Atos it already is.

## Usage

```
./trackver GARANCE            # storm name, any case
./trackver 2025054S17051      # or the IBTrACS storm id (SID)
./trackver_mars DUJUAN        # the same, with the forecasts from MARS
```

```
The storm was observed from 2025-02-23 06 to 2025-03-04 18 UTC; first landfall 2025-02-28 06 UTC.
Forecast runs:

   #  base time (UTC)   range   observed up to   steps with obs   landfall at
   ...
  10  2025-02-25 12     240 h   T+174 h            30            T+66 h
  11  2025-02-25 18     144 h   T+144 h            25            T+60 h
   ...

Which run (number)? 10
```

`./trackver_mars` adds a `models` column saying which forecasts each run will have:

```
   #  base time (UTC)   models     range   observed up to   steps with obs
   ...
  11  2026-09-18 00     IFS+AIFS   240 h   T+108 h            19
  12  2026-09-18 06     IFS+AIFS   240 h   T+102 h            18
   ...
```

`range` is the longest of the run's forecasts, up to `--max-hours`: the IFS 06 and 18 UTC runs
stop at 144 h, the 00 and 12 UTC runs and all AIFS runs go to 360 h. Each plot shows its own
model's range.

| option | meaning |
| --- | --- |
| `--run YYYYMMDDHH` | the run to plot, instead of choosing it from the list (e.g. for batch jobs) |
| `--max-hours N` | last forecast step to plot (default 240) |
| `--outdir DIR` | where to save the plots (default: `plots/`) |
| `--no-show` | save the plots without showing them |

```
./trackver_mars DUJUAN --run 2026091800 --no-show
```

The plots are saved in `plots/` and then shown on screen. If there is no display, they are only
saved. `<DTG>` is the run's base time, `YYYYMMDDHH`, and `<model>` is `ifs` or `aifs`:

```
./trackver        plots/ver_meteogram_<STORMNAME>_<DTG>.png           e.g. plots/ver_meteogram_GARANCE_2025022512.png
                  plots/ver_plumes_<STORMNAME>_<DTG>.png              e.g. plots/ver_plumes_GARANCE_2025022512.png
./trackver_mars   plots/ver_meteogram_<model>_<STORMNAME>_<DTG>.png   e.g. plots/ver_meteogram_aifs_DUJUAN_2026091800.png
                  plots/ver_plumes_<model>_<STORMNAME>_<DTG>.png      e.g. plots/ver_plumes_ifs_DUJUAN_2026091800.png
```

## Notebooks

`trackver.ipynb` (S3) and `trackver_mars.ipynb` (MARS, IFS and AIFS) go through the same steps
one cell at a time and show the plots in the notebook instead of saving them. Set the storm and
the run in the first code cell, then run all cells:

```
jupyter lab trackver.ipynb
```

```python
STORM = "GARANCE"       # if the name was reused, the storm observed at RUN is used
RUN = "2025022512"      # base time of the run, YYYYMMDDHH
```

Each notebook loads its program as a module, so the commands and the notebooks always behave
the same way.
