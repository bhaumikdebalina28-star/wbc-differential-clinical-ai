# WBC Differential — AI-Assisted Classification with Clinical Validation Framework

An open-source machine learning repository benchmarking the architectural workflow of digital hematology applications, specifically inspired by the core pipeline concepts of SigTuple's Shonit™ application. This project automates the classification of white blood cell (WBC) types from peripheral blood smear microscopy images using Transfer Learning, wrapped inside a robust clinical validation framework.

## 🔬 Core Objective
Rather than treating this solely as an isolated classification task, this project evaluates the underlying neural network through the lens of a regulated Software as a Medical Device (SaMD) product. The implementation supplements traditional metrics (such as confusion matrices) with an Intended Use Statement, an FMEA-based Risk Analysis, and a Human-in-the-Loop deployment protocol.

## 📂 Project Structure

```
wwbc-project/├── README.md                    <- System documentation and local environment setup├── requirements.txt              <- Python environment dependencies├── VALIDATION_FRAMEWORK.md       <- Clinical validation documentation (populated post-training)├── notebook/│   └── wbc_classification.ipynb  <- Data pipeline, model training, evaluation, and Grad-CAM interpretability└── data/                          <- Local dataset directory (excluded from version control via .gitignore)


## 🛠️ Local Environment Setup

### 1. Initialize Virtual Environment
Execute the following commands in the terminal inside the project root directory to create and activate an isolated Python environment:

```bash
python -m venv venv

# Windows (Command Prompt / PowerShell):
venv\Scripts\activate

# macOS / Linux:
source venv/bin/activate
```

### 2. Install Project Dependencies
Ensure `pip` is updated, then install the required computer vision and deep learning packages:
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### 3. Notebook Kernel Configuration
Ensure the "Jupyter" extension (by Microsoft) is installed in your development environment. When launching `notebook/wbc_classification.ipynb`, select the newly created `venv` environment as the active execution kernel.

### 4. Dataset Acquisition & Ingestion
The pipeline utilizes the **Blood Cell Images** dataset compiled by Paul Mooney on Kaggle. 

1. Authenticate via the Kaggle API by downloading the configuration token (`kaggle.json`) from the Kaggle Account Settings panel.
2. Initialize the automated download by placing the token in the corresponding system path:
   - **Windows:** `C:\Users\<Username>\.kaggle\kaggle.json`
   - **macOS/Linux:** `~/.kaggle/kaggle.json`
3. Download the target archive through the command-line interface:
   ```bash
   kaggle datasets download -d paultimothymooney/blood-cells
   ```
4. Extract the compressed file contents into the local `data/` directory:
   ```bash
   # Windows (PowerShell):
   Expand-Archive blood-cells.zip -DestinationPath data

   # macOS / Linux:
   unzip blood-cells.zip -d data
   ```
5. Confirm the extracted directory matches the following path convention:
   `data/dataset2-master/dataset2-master/images/TRAIN/`

---

## 📈 Expected Model Outputs & Evaluation
The deep learning system evaluates morphological variances across four distinct leukocyte populations: Eosinophils, Lymphocytes, Monocytes, and Neutrophils. Execution of the notebook yields the following analytical artifacts:

* **Classification Report:** Multi-class precision, recall, and F1-scores to evaluate clinical sensitivity and specificity boundaries.
* **Confusion Matrix:** Heatmap matrix isolating persistent classification friction points (e.g., differentiating atypical Monocytes from large Lymphocytes).
* **Grad-CAM Visualizations:** Explainable AI (XAI) class activation mapping overlaid onto input microscopy frames. These confirm that the convolutional layers converge on valid clinical markers (e.g., nuclear segmentation, granular density) rather than image artifacts, background noise, or slide staining anomalies.

---

## ⚠️ Clinical Engineering Limitations
* **Ground Truth Dependency:** Dataset annotations are accepted as absolute truth for this baseline, whereas real-world deployment requires multi-pathologist consensus verification to mitigate label noise.
* **Stain & Hardware Variance:** The underlying dataset is single-source. True production validation requires cross-scanner, cross-site, and multi-stain verification to establish robust model generalization.
* **Scope Definition:** This repository acts strictly as an educational proof-of-concept for practicing clinical SaMD design and must not be utilized for live diagnostic workflows.

---

## ⚖️ Disclaimer
This is an independent educational and research project designed to simulate medical machine learning workflows. It is inspired by public product descriptions of SigTuple's Shonit™ application. This project has no official affiliation, endorsement, or connection with SigTuple Technologies Pvt. Ltd., and does not utilize any proprietary code, data, or intellectual property belonging to the company.

## 📝 License
Distributed under the MIT License. See `LICENSE` for further details.