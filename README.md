
---

# MLOps DVC Experiment Pipeline Practice

## Summary Practice

This repository (`git@github:luisliborio/mlops-pipeline.git`) was used as a practice lecture for the MLops course on 13/03/2026. It demonstrates the implementation of an MLOps pipeline using DVC. We first started manually executing Python scripts with hardcoded parameters and then transitioned to an automated, parameterized, and version-controlled experiment pipeline with remote cloud storage.

Objective conclusion: Running ML experiments without pipelining automation can easily become a cumbersome task hard to track. You can easily get lost with the number of parameters, hyperparameters, metrics, models, data versions, etc. DVC is a straightforward tool that helps automating and tracking the MLops life cycle.

## Pipeline Evolution and Methodology

### 1. Initial Setup and Manual Execution

* **Repository Creation:** Initialized the GitHub repository `luisliborio/mlops-pipeline`.
* **Base Scripts Integration:** Added the foundational pipeline scripts to the `src/` directory:

    * `data_ingestion.py`: Fetches and splits the dataset.
    
    * `data_preprocessing.py`: Cleans and normalizes text data using NLTK.


    * `feature_engineering.py`: Vectorizes text using TF-IDF.
    * `model_building.py`: Trains a RandomForestClassifier.
    * `model_evaluation.py`: Evaluates model performance.


* **Manual Run:** Executed the scripts sequentially as a standard, non-automated Machine Learning experiment. At this stage, variables such as `test_size=0.2` and `n_estimators=25`  were hardcoded directly within the scripts.


* **Version Control:** Updated `.gitignore` to exclude raw data, models, and local reports before adding, committing, and pushing the initial codebase to Git.

### 2. DVC Pipeline Initialization

* **Pipeline Definition:** Created the initial `dvc.yaml` file to define the execution stages (`data_ingestion`, `data_preprocessing`, `feature_engineering`, `model_building`, `model_evaluation`).
 
**Parameter Isolation:** Initially, the `params:` definitions in `dvc.yaml` were left commented out, as the scripts were not yet configured to parse external parameter files.


* **First DVC Run:** Initialized DVC (`dvc init`) and executed the pipeline successfully using `dvc repro`. Pushed the environment configuration to Git.

### 3. Parameterization


**Externalizing Configurations:** Created a `params.yaml` file to centralize hyperparameters.


* **Script Modification:** Refactored the `.py` scripts to dynamically load parameters. Introduced the `load_params()` function utilizing `yaml.safe_load` to replace previously hardcoded values (e.g., `test_size`, `max_features`, `n_estimators`).
* **DVC Integration:** Uncommented the `params:` tracking blocks inside `dvc.yaml` to allow DVC to detect changes in the parameter configurations and trigger pipeline re-runs accordingly.

### 4. Experiment Tracking with DVCive

* **Setup:** Installed the `dvclive` library to monitor and log experiment metrics.
* **Implementation:** Modified `model_evaluation.py` to include the `Live(save_dvc_exp=True)` context manager. Configured it to log model metrics and parameters.
* **Experimentation:** * Ran multiple iterations using `dvc exp run` while altering hyperparameters in `params.yaml`.
* Utilized `dvc exp show` and the DVC VS Code extension to visually compare the outputs of different runs.
* Managed the workspace by deleting obsolete runs with `dvc exp remove`.
* Compared specific iterations using `dvc params diff`, `dvc metrics diff`, and plotted some comparative graphs using `dvc plots diff -x recall -y precision`.



### 5. Remote Storage Configuration

* **Storage Setup:** Initially attempted to configure Backblaze as the remote storage provider. **However, I run into authentication constraints**, and changed to GDrive + gcp.
* **Authentication & Sync:** Added and authenticated the gdrive remote within DVC.
* **Data Push:** Secured sensitive authentication credentials by removing them from Git tracking, then successfully pushed the versioned data artifacts using `dvc commit` and `dvc push`.

---

# dvc exp show --md > best_exp.md

| Experiment   | Created   | reports/metrics.json:accuracy   | reports/metrics.json:precision   | reports/metrics.json:recall   | auc     | dvclive/metrics.json:accuracy   | dvclive/metrics.json:precision   | dvclive/metrics.json:recall   | params.yaml:data_ingestion.test_size   | params.yaml:feature_engineering.max_features   | params.yaml:model_building.n_estimators   | params.yaml:model_building.random_state   | dvclive/params.yaml:data_ingestion.test_size   | dvclive/params.yaml:feature_engineering.max_features   | dvclive/params.yaml:model_building.n_estimators   | dvclive/params.yaml:model_building.random_state   | data/interim                         | data/processed                       | data/raw                             | models/model.pkl                 | src/data_ingestion.py            | src/data_preprocessing.py        | src/feature_engineering.py       | src/model_building.py            | src/model_evaluation.py          |
|--------------|-----------|---------------------------------|----------------------------------|-------------------------------|---------|---------------------------------|----------------------------------|-------------------------------|----------------------------------------|------------------------------------------------|-------------------------------------------|-------------------------------------------|------------------------------------------------|--------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|--------------------------------------|--------------------------------------|--------------------------------------|----------------------------------|----------------------------------|----------------------------------|----------------------------------|----------------------------------|----------------------------------|
| workspace    | -         | 0.97278                         | 0.98261                          | 0.81884                       | 0.97358 | 0.97278                         | 0.98261                          | 0.81884                       | 0.18                                   | 400                                            | 200                                       | 2                                         | 0.18                                           | 400                                                    | 200                                               | 2                                                 | 5583b02cb170b80e77a0c3d8085e0cba.dir | 267f1028484b14f53d0a4028513f15d0.dir | 559ae6a6545db557c5378bbdd5246e96.dir | 3a9061eb1f43f3bb92527cc4125c58c3 | e0b5bb0235439fcc0b34a231792b2236 | 66a41bf489676519b3def412169410bf | 76d80040dafda494d9d3fd38793c24c6 | 29d15ba57dccf7429a737debcfb35d9d | bfe74557e498cda8091771252d7bbd9b |
| main         | 05:23 AM  | 0.97278                         | 0.98261                          | 0.81884                       | 0.97358 | 0.97278                         | 0.98261                          | 0.81884                       | 0.18                                   | 400                                            | 200                                       | 2                                         | 0.18                                           | 400                                                    | 200                                               | 2                                                 | 5583b02cb170b80e77a0c3d8085e0cba.dir | 267f1028484b14f53d0a4028513f15d0.dir | 559ae6a6545db557c5378bbdd5246e96.dir | 3a9061eb1f43f3bb92527cc4125c58c3 | e0b5bb0235439fcc0b34a231792b2236 | 66a41bf489676519b3def412169410bf | 76d80040dafda494d9d3fd38793c24c6 | 29d15ba57dccf7429a737debcfb35d9d | bfe74557e498cda8091771252d7bbd9b |

