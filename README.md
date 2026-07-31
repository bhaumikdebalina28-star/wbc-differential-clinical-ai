# WBC Differential — AI-Assisted Classification with Clinical Validation Framework

An open-source project inspired by / benchmarking the architectural workflow of SigTuple's Shonit™ application : classifying white blood cell types
from microscopy images using transfer learning, wrapped in a clinical validation framework
(intended use, risk analysis, validation protocol, human-in-the-loop deployment plan).

## Why this project
Rather than just training a classifier and reporting accuracy, this project treats the model
the way a regulated diagnostic AI would need to be treated before it ships — with an intended
use statement, a risk analysis, and a validation protocol, not just a confusion matrix.

## Project structure
```
wbc-project/
├── README.md                    <- you are here
├── requirements.txt              <- Python dependencies
├── VALIDATION_FRAMEWORK.md       <- the clinical validation document (fill in after training)
├── notebook/
│   └── wbc_classification.ipynb  <- data, model, training, evaluation, Grad-CAM
└── data/                          <- dataset goes here (not committed to git)
```

## Setup (VS Code, local machine)

### 1. Create and activate a virtual environment
Open the VS Code terminal (`` Ctrl+` `` on Windows/Linux, `` Cmd+` `` on Mac) in this folder:

```bash
python -m venv venv

# Windows:
venv\Scripts\activate

# Mac/Linux:
source venv/bin/activate
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. Install the Jupyter extension in VS Code
Go to the Extensions panel (left sidebar) and install "Jupyter" (by Microsoft), if not already installed.
Then open `notebook/wbc_classification.ipynb` — VS Code will let you run cells directly, select
`venv` as the kernel when prompted (top-right of the notebook).

### 4. Get the dataset (Kaggle "Blood Cell Images" by Paul Mooney)
1. Create a free Kaggle account if you don't have one: kaggle.com
2. Go to kaggle.com → your profile picture → Settings → API → "Create New Token".
   This downloads a file called `kaggle.json`.
3. Place `kaggle.json` in:
   - Windows: `C:\Users\<you>\.kaggle\kaggle.json`
   - Mac/Linux: `~/.kaggle/kaggle.json`
4. In the terminal (with venv active), run:
   ```bash
   kaggle datasets download -d paultimothymooney/blood-cells
   ```
5. Unzip it into the `data/` folder in this project:
   ```bash
   # Windows (PowerShell):
   Expand-Archive blood-cells.zip -DestinationPath data

   # Mac/Linux:
   unzip blood-cells.zip -d data
   ```
6. You should end up with a folder structure like:
   ```
   data/dataset2-master/dataset2-master/images/TRAIN/EOSINOPHIL/...
   data/dataset2-master/dataset2-master/images/TRAIN/LYMPHOCYTE/...
   data/dataset2-master/dataset2-master/images/TRAIN/MONOCYTE/...
   data/dataset2-master/dataset2-master/images/TRAIN/NEUTROPHIL/...
   ```
   (Double-check the exact path after unzipping — the dataset has a couple of nested folders;
   the notebook's first cell lets you set this path once.)

### 5. Run the notebook
Open `notebook/wbc_classification.ipynb` in VS Code and run cells top to bottom. Each section
is labeled — data loading, EDA, model, training, evaluation, Grad-CAM.

## After training: fill in VALIDATION_FRAMEWORK.md
Once you have real metrics (confusion matrix, per-class precision/recall), go fill in the
bracketed placeholders in `VALIDATION_FRAMEWORK.md` with your actual numbers and observations.
That document is the part that differentiates this project for a clinical-AI-adjacent role —
don't skip it even if you're short on time.

## Honest limitations (worth saying out loud in an interview)
- Dataset labels are treated as ground truth, but in reality this would need multi-pathologist
  consensus labeling — a real limitation, not glossed over.
- This is a small, single-source public dataset — no cross-scanner or cross-stain validation,
  which real deployment would require.
- This is a learning project, not a validated clinical tool — presented as a demonstration of
  how you *think* about validation, not as a finished product.
