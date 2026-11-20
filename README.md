# ST-GCN-LSTM daily crime count prediction, Chicago census tracts

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.24122178.svg)](https://doi.org/10.5281/zenodo.24122178)

Code and recorded results from my MSc dissertation at the University of Hull, *Spatio-Temporal Graph Convolutional Networks with Long Short-Term Memory for Urban Crime Prediction in Chicago* (May 2026). The models predict the daily crime count of each of 878 Chicago census tracts. The main model, ST-GCN-LSTM, combines a graph attention network over a k-nearest-neighbour tract graph with a bidirectional LSTM over a 30-day lookback. It is compared with four baselines on the same input and output: GCN, LSTM, CNN and CNN+LSTM.

Author: Sodiq Olatunji Azeez

The whole pipeline is one notebook, `ST_GCN_LSTM_Crime_Prediction_Notebook...ipynb`. It loads the data, builds the tract graph and the daily grid, splits the days in time order, trains the five models and reports the test metrics.

## Data

No data is stored in this repository. Download both files from the City of Chicago Data Portal.

- Crime records: "Crimes - 2001 to Present", https://data.cityofchicago.org/Public-Safety/Crimes-2001-to-Present/ijzp-q8t2 , exported as CSV. Set its path in the cell that calls `load_crime_data`.
- Census tract boundaries: "Boundaries - Census Tracts - 2000", which the notebook reads directly from https://data.cityofchicago.org/resource/4hp8-2i8z.geojson?$limit=50000 .

The study period is 1 January 2021 to 31 December 2023.

## Setup

Python 3.13 with the CPU build of PyTorch:

```
python -m venv .venv
.venv\Scripts\pip install torch==2.9.1 --index-url https://download.pytorch.org/whl/cpu
.venv\Scripts\pip install -r requirements.txt
```

On a Mac or Linux, use `.venv/bin/pip`. Then open the notebook in Jupyter, set the crime file path and run all cells. The notebook writes `data/`, `models/` and `outputs/` in the working folder.

## Design

- Time-ordered split: training 1 January 2021 to 31 October 2023 (1,004 samples after the 30-day lookback), validation November 2023, test December 2023.
- Test set: 31 days by 878 tracts, 27,218 predictions per model.
- One fixed input and output contract for all five models, so they are scored on identical data.
- The baselines are trained with a smooth L1 (Huber) loss and ST-GCN-LSTM with mean squared error, as described in the dissertation.
- A lookback sweep scores ten windows from 1 to 180 days; the reported models use 30.

## Results on the test month

| Model | MAE | RMSE | R2 |
|---|---|---|---|
| GCN | 0.7521 | 1.0965 | 0.1023 |
| LSTM | 0.6613 | 0.9561 | 0.3173 |
| CNN | 0.6673 | 0.9491 | 0.3273 |
| CNN+LSTM | 0.6637 | 0.9498 | 0.3264 |
| ST-GCN-LSTM | 0.7456 | 1.0441 | 0.1859 |

The figures are the notebook's saved output. Each model is trained once and no random seed is fixed, so a rerun can differ slightly.

## Note on the last day

`load_crime_data` keeps records with `date <= '2023-12-31'`, which compares against midnight at the start of that day, so 31 December 2023 holds only the records stamped exactly 00:00:00. The saved results are unchanged.

## Licence and citation

MIT licence, see `LICENSE`. To cite this work, use the DOI above or `CITATION.cff`.
