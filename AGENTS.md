# AGENTS.md

## 1. Project Context

This repository is primarily used for AI/ML research, NLP experiments, paper implementation, and machine learning competitions.

The user is a graduate student working on AI/NLP research.

Prioritize:

1. Correctness
2. Experimental reproducibility
3. Clear and understandable code
4. Fast iteration
5. Easy modification of experiments

Do not over-engineer research code.

Avoid unnecessary abstractions, excessive class hierarchies, complex design patterns, or production-oriented architecture unless explicitly requested.

The code should be understandable and modifiable by a graduate student conducting experiments.

---

## 2. Before Starting Any Task

Before modifying code:

1. Inspect the current repository structure.
2. Read relevant existing files.
3. Understand the current implementation before proposing changes.
4. Check the current Python environment and installed dependencies when relevant.
5. Check whether a `.venv` virtual environment already exists.
6. Check existing configuration files and experiment conventions.
7. Check Git status before making broad or destructive changes.

Do not assume that a missing feature requires creating a new architecture.

Prefer modifying the existing implementation when reasonable.

Do not perform large-scale refactoring unless explicitly requested or clearly necessary.

If a requested change may invalidate existing experiments or outputs, explain the risk before proceeding.

---

## 3. Python Environment

Use a project-local Python virtual environment named:

`.venv`

All Python commands and package installations should use the project's `.venv`.

On Windows, prefer explicit execution such as:

`.venv\Scripts\python.exe`

and:

`.venv\Scripts\python.exe -m pip`

Do not install Python packages globally unless explicitly requested.

If `.venv` does not exist and Python execution is required:

1. Check the installed Python version.
2. Create `.venv`.
3. Upgrade pip.
4. Install only the dependencies required by the project.

Before installing a package, check whether it is already installed and whether the project specifies a required version.

Avoid unnecessary package installations.

Do not install Anaconda or create Conda environments unless explicitly requested.

---

## 4. GPU and PyTorch

The development machine may contain an NVIDIA GPU.

For PyTorch projects:

1. Check `nvidia-smi` when GPU availability is relevant.
2. Check `torch.cuda.is_available()`.
3. Check `torch.version.cuda`.
4. Check the detected GPU name.
5. Prefer CUDA execution when a compatible NVIDIA GPU is available.
6. Make device selection explicit in the code.

Preferred pattern:

```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
```

Move both models and tensors to the correct device.

Do not assume that installing the system-wide CUDA Toolkit is required for normal PyTorch usage.

Do not install the full CUDA Toolkit unless:

* custom CUDA compilation is required,
* a CUDA extension explicitly requires it,
* or the user explicitly requests it.

When installing PyTorch, use an official PyTorch CUDA build compatible with the current environment.

Do not silently replace a working CUDA-enabled PyTorch installation with a CPU-only build.

After GPU-related setup, verify the environment with an actual CUDA tensor operation.

When practical, report:

* PyTorch version
* PyTorch CUDA build version
* CUDA availability
* GPU name
* GPU count

---

## 5. Research Code Principles

This is research-oriented code.

Prefer simple and explicit implementations over clever abstractions.

Code should make the experimental logic easy to inspect.

Important experimental decisions should be visible in code or configuration.

Examples include:

* model name
* learning rate
* batch size
* number of epochs
* optimizer
* scheduler
* random seed
* train/validation split
* maximum sequence length
* loss function
* evaluation metric
* checkpoint selection criteria

Avoid unexplained magic numbers.

Use clear variable names.

Add comments for important research logic, especially when the reason for an implementation choice is not obvious.

Do not add comments that merely repeat the code.

---

## 6. Reproducibility

Experimental reproducibility is important.

Set random seeds when randomness is involved.

For example, consider:

* Python `random`
* NumPy
* PyTorch CPU
* PyTorch CUDA

When appropriate, create a reusable seed function.

Example:

```python
def set_seed(seed: int = 42):
    import random
    import numpy as np
    import torch

    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)

    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)
```

Be aware that deterministic settings may reduce performance.

Do not enable expensive deterministic behavior without explaining the tradeoff when it materially affects training speed.

Record important experiment settings.

Prefer configuration files or a clearly defined configuration section when multiple experiments are expected.

---

## 7. Data Leakage

Always consider data leakage.

Before training or validation logic is finalized, inspect for potential leakage involving:

* target variables
* future information
* timestamps
* sequence ordering
* duplicated samples
* entity overlap
* user overlap
* group overlap
* preprocessing fitted on the full dataset
* normalization using validation or test statistics
* feature engineering using future observations

For sequential or temporal prediction tasks, never randomly split data without first checking whether temporal ordering matters.

For grouped data, consider group-aware splitting.

Fit preprocessing components using training data only when required.

If a possible leakage issue is found, explicitly report it.

Do not silently continue with a suspicious validation setup.

---

## 8. Training and Validation

Training code should clearly separate:

* training
* validation
* inference

Validation metrics should match the competition or research objective whenever possible.

Do not optimize a convenient metric if the actual evaluation metric is different without explaining the mismatch.

Track relevant training information such as:

* training loss
* validation loss
* primary validation metric
* learning rate
* epoch

When appropriate, save the best checkpoint based on the primary validation metric.

Clearly define whether a higher or lower metric is better.

Avoid evaluating only on the training set.

---

## 9. Experiment Management

Experiments should be easy to compare.

When multiple experiments are expected, record at least:

* experiment name or ID
* timestamp when useful
* model
* hyperparameters
* seed
* validation score
* relevant notes

Prefer simple formats such as:

* CSV
* JSON
* YAML

Do not introduce a heavy experiment tracking platform unless explicitly requested.

If the project already uses MLflow, Weights & Biases, TensorBoard, or another tracking system, follow the existing convention.

Do not overwrite previous experiment results unless explicitly requested.

Use unique experiment directories or identifiers when appropriate.

---

## 10. Project Structure

Do not restructure an existing repository unnecessarily.

For a new small or medium research project, a structure similar to the following is preferred:

```text
project/
├── AGENTS.md
├── README.md
├── requirements.txt
├── .gitignore
├── configs/
├── data/
├── notebooks/
├── src/
├── scripts/
├── outputs/
└── .venv/
```

Possible responsibilities:

* `configs/`: experiment configurations
* `data/`: local datasets
* `notebooks/`: exploration and analysis
* `src/`: reusable implementation
* `scripts/`: training, validation, and inference entry points
* `outputs/`: experiment results and checkpoints

For a very small experiment, keep the structure simpler.

Do not create directories that have no immediate purpose.

---

## 11. Notebooks

Jupyter notebooks are acceptable for:

* exploratory data analysis
* dataset inspection
* visualization
* quick experiments
* debugging model behavior

Avoid putting the entire long-term training pipeline only in a notebook when the experiment is expected to be repeated.

Move reusable logic into Python modules when appropriate.

Keep notebooks readable and avoid excessive hidden state.

When notebook execution order matters, make the dependency clear.

---

## 12. Dependency Management

Maintain project dependencies.

Use `requirements.txt` unless the project already uses another dependency management format.

Only add packages that are actually required.

Avoid blindly running:

`pip freeze > requirements.txt`

if it would add unrelated environment packages to a carefully maintained dependency file.

When creating a new requirements file from a clean project environment, `pip freeze` may be used.

For research reproduction, pin important package versions when version differences may affect results.

Pay particular attention to versions of:

* torch
* torchvision
* transformers
* accelerate
* datasets
* numpy
* pandas
* scikit-learn
* xgboost
* lightgbm

Do not arbitrarily upgrade major dependencies in an existing project.

---

## 13. Git and Large Files

Ensure `.gitignore` excludes local or generated files when appropriate.

Common exclusions include:

```gitignore
.venv/
__pycache__/
*.pyc
.env

data/
outputs/
checkpoints/
wandb/

*.pt
*.pth
*.ckpt
```

Do not blindly add these rules if the repository intentionally tracks specific files in these locations.

Inspect the existing repository first.

Do not commit:

* API keys
* access tokens
* passwords
* `.env` files
* private credentials

Do not commit large datasets or model checkpoints unless the repository explicitly requires them.

Before broad Git changes, inspect the current Git status.

Do not reset, discard, or overwrite uncommitted user changes without explicit permission.

Do not force push unless explicitly requested.

---

## 14. Secrets and Credentials

Never hardcode secrets.

Examples:

* OpenAI API keys
* Hugging Face tokens
* Dacon credentials
* WandB API keys
* cloud credentials

Use environment variables when credentials are required.

Example:

```python
import os

api_key = os.getenv("OPENAI_API_KEY")
```

If a `.env` file is used, ensure `.env` is ignored by Git.

Do not print complete secrets in logs or terminal output.

---

## 15. Code Modification Policy

When implementing a requested feature:

1. Identify the minimum relevant files.
2. Understand the existing code path.
3. Make the smallest reasonable change.
4. Preserve existing behavior unless the task requires changing it.
5. Test the modified path.
6. Report what changed.

Avoid unrelated cleanup during feature work.

Do not rename files, functions, classes, or variables across the repository unless necessary.

Do not delete existing code simply because another implementation appears cleaner.

If substantial refactoring is recommended, explain the reason and ask before performing a broad rewrite unless the user explicitly requested refactoring.

---

## 16. Testing and Verification

Do not stop after writing code.

After changes, perform the strongest practical verification available.

At minimum, consider:

1. Syntax checks
2. Import checks
3. Relevant unit tests if present
4. Small execution tests
5. Smoke tests

For machine learning training code, do not immediately run a long full training job unless requested.

First perform a smoke test using:

* a small subset of data
* a small number of batches
* one epoch or fewer
* reduced workers when debugging

Verify that:

* data loading works
* tensor shapes are correct
* forward pass works
* loss is computed
* backward pass works when training
* optimizer step works
* validation runs
* checkpoint logic does not immediately fail

For GPU code, verify at least one actual CUDA operation when possible.

Do not claim that code works if it was not executed.

Clearly distinguish:

* implemented
* syntax checked
* import checked
* smoke tested
* fully trained
* fully evaluated

---

## 17. Error Handling and Debugging

When an error occurs:

1. Read the complete error message and traceback.
2. Identify the likely root cause.
3. Inspect the relevant code and environment.
4. Make a targeted fix.
5. Run the failing command again.
6. Confirm whether the original error is resolved.

Do not repeatedly apply speculative fixes without checking results.

Do not hide exceptions with broad patterns such as:

```python
try:
    ...
except Exception:
    pass
```

unless there is a specific and justified reason.

Prefer fixing the root cause.

If an environment or dependency issue is suspected, inspect actual versions before changing packages.

---

## 18. Paper Implementation

When implementing a research paper:

1. Identify the exact paper version.
2. Check whether official code exists.
3. Read the relevant methodology sections.
4. Identify the main contribution.
5. Separate paper-specified details from implementation assumptions.
6. Match the original evaluation setup when reproduction is the goal.

Do not claim that an implementation exactly reproduces a paper unless the implementation and experiment setup actually match.

Clearly label assumptions or missing details.

If the paper is ambiguous, explain the ambiguity.

When modifying a paper method, preserve a clear baseline implementation when practical.

The experimental comparison should make it possible to distinguish:

* baseline
* paper method
* proposed modification

---

## 19. Competition Projects

For machine learning competitions:

1. Read and understand the evaluation metric.
2. Inspect train and test schemas.
3. Check target distribution.
4. Check missing values.
5. Check duplicates.
6. Check sequence, temporal, entity, or group structure.
7. Investigate possible leakage.
8. Build a simple baseline before complex models.
9. Establish a reliable local validation strategy.
10. Compare experiments systematically.

Do not assume that a more complex deep learning model will outperform a simpler baseline.

When applicable, compare against:

* heuristic baseline
* statistical baseline
* tree-based model
* simple neural baseline

Competition leaderboard score should not be treated as the only evidence of generalization.

Pay close attention to public leaderboard overfitting.

---

## 20. Performance Optimization

Do not optimize prematurely.

First ensure correctness.

For GPU workloads, investigate performance using actual evidence.

Potential areas include:

* batch size
* data loading
* `num_workers`
* pinned memory
* mixed precision
* gradient accumulation
* unnecessary CPU-GPU transfers
* sequence padding
* model size

Use mixed precision when appropriate.

For modern PyTorch code, prefer supported AMP APIs.

Do not apply performance optimizations that change model behavior without explaining the effect.

---

## 21. Communication Style

When reporting work, be concise and specific.

Do not provide a long generic explanation unless requested.

After completing a task, summarize:

### Changed Files

List the files that were created or modified.

### Key Changes

Explain the important implementation changes.

### How to Run

Provide the exact command needed to run the relevant code.

### Verification

State exactly what was tested and the result.

### Notes

Mention important assumptions, risks, leakage concerns, or unfinished work.

If an experiment was not fully trained, explicitly say so.

If a result is uncertain, do not present it as confirmed.

---

## 22. User Intent

The user is learning AI/NLP research while actively implementing models and experiments.

When a technically important decision is made, briefly explain the reason.

Examples:

* why a validation split was chosen
* why a model architecture is appropriate
* why a package version matters
* why a CUDA build was selected
* why a possible leakage issue exists

However, do not turn every code change into a tutorial.

Prioritize completing the requested task while making important research decisions understandable.

When several implementation choices are reasonable, prefer the simplest strong baseline unless the user explicitly requests a more advanced approach.
