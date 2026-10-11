# DocuShield AI 🛡️
### AI-Based Fake and Manipulated Document Detection & Localization

DocuShield AI is an AI-assisted document forensics project designed to identify potential signs of digital manipulation in document images and PDFs, highlight suspicious regions, and explain the evidence that may warrant further review.

Documents are increasingly shared digitally for academic verification, recruitment, financial transactions, and administrative processes. Changes to names, dates, marks, amounts, signatures, stamps, or other content can be difficult to spot by visual inspection alone. DocuShield AI aims to make document review more systematic by combining computer vision, image-forensic signals, OCR, and PDF structural analysis.

> **Important:** DocuShield AI is intended to support human review, not to make a definitive legal or factual determination that a document or person is fraudulent. An automated result is not proof of fraud.

---

## Table of Contents
- [Problem Statement](#-problem-statement)
- [What We Aim to Build](#-what-we-aim-to-build)
- [Scope](#-scope)
- [Proposed Approach](#-proposed-approach)
- [Technology Stack](#-technology-stack)
- [System Architecture](#-system-architecture)
- [Dataset Strategy](#-dataset-strategy)
- [Evaluation Plan](#-evaluation-plan)
- [Expected Features](#-expected-features)
- [Project Roadmap](#-project-roadmap)
- [Repository Structure](#-repository-structure)
- [Getting Started](#-getting-started)
- [Limitations and Responsible Use](#-limitations-and-responsible-use)
- [Project Status](#-project-status)
- [Contributing](#-contributing)
- [License](#-license)

## 🎯 Problem Statement

Digital documents may be manipulated by inserting, deleting, replacing, or duplicating content while preserving an appearance that looks convincing to a human reviewer. Examples include altered names, dates, marks, financial amounts, text, images, signatures, and stamps.

Manual verification is time-consuming and may miss subtle changes. The goal of DocuShield AI is to build an AI-assisted forensic pipeline that:

1. Analyzes uploaded PDF and image documents for potential signs of digital manipulation.
2. Identifies and highlights regions that appear suspicious.
3. Combines visual-forensic indicators with OCR-derived text and layout information, and PDF structural signals when available.
4. Explains which indicators contributed to the result and communicates uncertainty.
5. Produces a report that helps a person decide what should be reviewed more closely.

The project focuses on **detecting potential digital manipulation in a document's digital representation**. It cannot, by image analysis alone, establish whether the information is factually true or whether a document was genuinely issued by an authority.

## 💡 What We Aim to Build

The intended system will accept a PDF, JPG, or PNG and run a sequence of analyses before displaying a result.

Planned outputs include:
- A document-level assessment such as **Potential Manipulation Detected**, **No Manipulation Detected by the Current Checks**, or **Inconclusive**.
- A heatmap or mask highlighting potentially manipulated regions, where the localization model supports it.
- OCR-extracted text and text coordinates, where available.
- Forensic indicators such as local noise, edge, texture, compression, or copy-move inconsistencies.
- PDF metadata and structural observations, when applicable.
- An explainable report containing the findings, model scores, limitations, and suggested manual-review steps.

These are intended features, not claims that the current implementation already supports them.

## 🔍 Scope

### Initial focus
The first version will prioritize:
- **Text alteration:** possible changes to names, dates, numbers, marks, amounts, or other text.
- **Content insertion or splicing:** possible insertion of text or visual elements.
- **Copy-move manipulation:** possible duplication of a region to replace, conceal, or repeat content.

### Possible future extensions
- Text erasure and background reconstruction.
- Signature and stamp manipulation.
- Photograph replacement.
- More detailed PDF object-level modification analysis.
- Generalization across document templates and document types.

Initial demonstration documents may include fictional invoices, marksheets, certificates, and administrative forms.

## 🧭 Proposed Approach

The project will be developed incrementally. Each module will be evaluated before it is integrated into the complete application.

### 1. Input validation and preprocessing
- Validate file type, file size, and page count.
- Render PDF pages to images at controlled resolutions.
- Normalize image inputs and handle rotation or perspective issues when appropriate.
- Preserve the original file and track coordinate transformations so detected regions can be mapped back to the source.

**Planned tools:** PyMuPDF, OpenCV, Pillow.

### 2. OCR and layout extraction
- Extract recognized text and word-level or character-level coordinates.
- Explore text alignment, spacing, and layout features that may help identify suspicious areas.
- Treat OCR as supporting evidence, not as an authenticity test.

**Candidate tools:** PaddleOCR or Tesseract. The final choice will be made after testing.

### 3. Visual-forensics analysis
Explore signals that may reveal inconsistencies between document regions:
- Error Level Analysis (ELA).
- Noise residuals and local texture.
- Edge inconsistencies.
- Compression artifacts.
- Copy-move similarities.
- Frequency-domain features.
- Deep-learning features for manipulation detection.

These signals are heuristics. For example, ELA differences can arise from ordinary editing or compression and are not proof of manipulation.

### 4. Document-level classification
Establish simple baselines before trying more complex models:
- A traditional machine-learning baseline, such as an SVM or Random Forest using selected features.
- A pretrained CNN baseline, such as ResNet or EfficientNet, fine-tuned if the data and compute budget allow.

The classifier will estimate whether the input resembles examples of manipulated or unmanipulated documents in the evaluation data. It will not, by itself, identify the exact manipulated pixels.

### 5. Suspicious-region localization
Investigate a segmentation model such as **U-Net** to produce a pixel-level probability map and predicted mask.

Training this component requires manipulated samples with corresponding ground-truth masks. The mask should identify the regions altered during the creation of each training sample.

### 6. PDF structural analysis
For PDFs, inspect available signals such as:
- Metadata.
- Embedded images.
- Text objects and their coordinates.
- Font information.
- Incremental updates or object-level inconsistencies, where detectable.
- Differences between extracted content and rendered appearance.

Metadata or structural anomalies may also result from legitimate editing, conversion, or export; they must not be treated as conclusive evidence.

### 7. Evidence fusion and uncertainty
Compare:
1. Visual-forensics-only analysis.
2. OCR/layout-based analysis.
3. A combined approach using available visual and OCR/layout signals.

Test whether fusion improves detection and localization. The system should account for missing or unreliable signals and be able to return an **Inconclusive** result when evidence is insufficient. Model scores will not automatically be presented as calibrated probabilities of fraud.

### 8. Explainable report and interface
A planned Streamlit interface will allow users to upload a supported file, inspect the analysis, view suspicious-region overlays, and review a summary of findings and limitations.

## 🧰 Technology Stack

The stack below is proposed. Some alternatives will be evaluated during the initial experiments; final choices may change based on dataset compatibility and results.

| Area | Planned technology | Purpose |
|---|---|---|
| Language | Python | Main implementation language |
| Deep learning | PyTorch | Classification and segmentation experiments |
| Traditional ML | scikit-learn | Feature-based baselines and evaluation utilities |
| Computer vision | OpenCV, Pillow | Image processing and visual-forensics features |
| PDF processing | PyMuPDF | PDF rendering and structural/text inspection |
| OCR | PaddleOCR or Tesseract | Text extraction and text-coordinate analysis |
| Data handling | NumPy, Pandas | Data preparation and analysis |
| Prototype UI | Streamlit | Interactive upload and report interface |
| Optional backend | FastAPI | API layer if the application later needs separation |
| Visualization | Matplotlib | Masks, heatmaps, and experiment plots |
| Experiment tracking | TensorBoard or equivalent | Training and experiment monitoring |
| Version control | Git, GitHub | Source control and collaboration |

## 🏗️ System Architecture

```text
PDF / JPG / PNG
      |
      v
File Validation & Preprocessing
      |
      +--------------------------+
      |                          |
      v                          v
OCR & Layout Extraction    Visual Forensics
      |                          |
      v                          v
Text / Geometry Features   Image Features / Model
      |                          |
      +------------+-------------+
                   |
                   v
        Evidence Fusion Layer
                   |
       +-----------+-----------+
       |                       |
       v                       v
Document-Level Result   Suspicious-Region Mask
       |                       |
       +-----------+-----------+
                   |
                   v
      Uncertainty & Report Generation
                   |
                   v
          Human Review Interface

PDF Structural Analysis is an additional branch for PDF inputs.
```

This is the proposed architecture; implementation details will be refined after dataset inspection and baseline experiments.

## 🗂️ Dataset Strategy

Dataset selection will be based on access, license, annotation quality, task fit, and reproducibility. Dataset availability and terms must be checked before use.

Potential research datasets to investigate:

- **DocTamper** — research dataset focused on tampered text in document images.  
  Paper: [CVPR 2023 paper](https://openaccess.thecvf.com/content/CVPR2023/html/Qu_Towards_Robust_Tampered_Text_Detection_in_Document_Image_New_Dataset_CVPR_2023_paper.html) · [Official repository](https://github.com/qcf-568/DocTamper)
- **SCUT-EnsExam** — investigate its relevance to examination-document tampering and the compatibility of its labels with the localization task.  
  [Repository](https://github.com/SCUT-DLVCLab/SCUT-EnsExam)
- **CASIA-DT** — investigate the exact release, annotations, access conditions, and usage terms before deciding whether it fits the project.
- **Custom synthetic dataset** — create fictional templates and programmatically generate altered versions with known pixel-level masks.

For each synthetic sample, the data pipeline should preserve the original, manipulated version, ground-truth mask, manipulation type, transformation/degradation details, and template identifier.

**Data-splitting rule:** Original documents and their manipulated variants must remain in the same train, validation, or test partition. This helps prevent near-duplicate leakage and overly optimistic results.

Do not commit restricted datasets or private identity documents to the public repository. Check the terms of use for every dataset before downloading, using, or redistributing it.

## 📏 Evaluation Plan

Performance will be assessed at both document and pixel levels. Accuracy alone is not sufficient.

### Document-level detection
- Precision, recall, and F1-score.
- Confusion matrix and false-positive rate.
- ROC-AUC where appropriate.
- Calibration metrics if probabilistic outputs are used.

### Pixel-level localization
- Pixel precision and recall.
- Pixel-level F1-score.
- Intersection over Union (IoU).
- Dice coefficient.

### Robustness and generalization
Evaluate, where the dataset permits, under:
- JPEG compression.
- Resizing and blur.
- Added noise.
- Low-resolution scanning.
- Camera photography and perspective distortion.
- Previously unseen templates or document types.

Results should be reported separately for different conditions. Experiments should use reproducible settings, document-level independent splits, appropriate baselines, and error analysis.

## ✨ Expected Features

- [ ] PDF, JPG, and PNG input validation.
- [ ] PDF rendering and image preprocessing.
- [ ] OCR text and coordinate extraction.
- [ ] Visual-forensics feature analysis.
- [ ] Document-level classification baseline.
- [ ] Pixel-level suspicious-region localization.
- [ ] PDF metadata and structural inspection.
- [ ] Fusion of visual and OCR/layout evidence.
- [ ] Uncertainty-aware, explainable report.
- [ ] Streamlit user interface.
- [ ] Robustness tests and reproducible evaluation.

The checklist reflects the intended roadmap, not completed functionality.

## 🗺️ Project Roadmap

1. **Feasibility and literature review** — inspect datasets, licenses, labels, and recent research.
2. **Data pipeline** — define a reproducible format and preprocessing workflow.
3. **Synthetic data generation** — create controlled examples and localization masks if needed.
4. **Baseline detection** — establish traditional-feature and CNN baselines.
5. **Localization** — train or evaluate a segmentation model using mask annotations.
6. **OCR and PDF analysis** — add text/layout and PDF structural signals.
7. **Evidence fusion** — compare visual-only, OCR/layout-only, and combined methods.
8. **Robustness and error analysis** — test degradations, unseen templates, false positives, and false negatives.
9. **Uncertainty and reporting** — investigate calibration, inconclusive outcomes, and evidence summaries.
10. **Application and documentation** — integrate the prototype, test it, and document reproducible results.

The first milestone is a working, measurable baseline—not a complete system in one step.

## 📁 Proposed Repository Structure

```text
docushield-ai/
├── app/                     # Streamlit interface
├── data/
│   ├── originals/           # Local data; generally not committed
│   ├── manipulated/         # Local data; generally not committed
│   └── masks/               # Ground-truth masks; subject to dataset terms
├── notebooks/
│   ├── dataset_analysis.ipynb
│   └── baseline_experiments.ipynb
├── src/
│   ├── preprocessing/
│   ├── ocr/
│   ├── forensics/
│   ├── models/
│   ├── evaluation/
│   └── reporting/
├── tests/
├── train.py
├── requirements.txt
├── .gitignore
└── README.md
```

The repository layout may evolve as the implementation develops. Avoid committing private documents, restricted datasets, secrets, large model weights, or generated files that should not be redistributed.

## 🚀 Getting Started

**Current status:** The project is in the planning and feasibility stage. A verified, end-to-end implementation and finalized dependency list are not yet available, so installation and run commands will be added once the initial code is in place.

### Prerequisites (planned)
- Python 3.10+ (confirm the exact version against selected libraries).
- Git.
- A CPU for initial experiments; a compatible GPU can help with deep-learning training.

### Planned setup

```bash
git clone <your-repository-url>
cd docushield-ai

python -m venv .venv
```

Activate the environment:

**Windows PowerShell**
```powershell
.\.venv\Scripts\Activate.ps1
```

**macOS / Linux**
```bash
source .venv/bin/activate
```

Once dependencies have been selected and `requirements.txt` has been created, install them with:

```bash
pip install -r requirements.txt
```

The application launch command will be documented after the first working prototype is implemented.

## ⚠️ Limitations and Responsible Use

- **Not proof of fraud:** A model prediction is an indicator for review, not a final verdict.
- **Manipulation vs. falsity:** A visually authentic document may contain false information, and a manipulated file may reflect legitimate editing. Image forensics alone cannot verify factual correctness or official issuance.
- **False positives and negatives:** Compression, scanning, printing, resizing, and ordinary editing can introduce artifacts or hide manipulation.
- **No guaranteed coverage:** The system will only be evaluated for the manipulation types and data distributions represented in testing.
- **Metadata limitations:** Missing or unusual metadata is not proof of manipulation.
- **Privacy:** Use fictional or authorized examples, minimize retention of uploaded documents, and do not expose sensitive information in logs or public demos.
- **Dataset compliance:** Follow the license and usage conditions of each dataset and model.

## 📌 Project Status

**Stage:** Planning, dataset feasibility, and research design.

The project name and proposed stack are working choices. Dataset selection, model architecture, and the research contribution will be finalized after feasibility checks and literature review. Any research novelty claim will be made only after comparison with relevant existing work.

## 🤝 Contributing

Ideas, issue reports, and suggestions are welcome. Contributions should prioritize reproducibility, clear evaluation, privacy, and responsible interpretation of forensic outputs.

## 📄 License

A repository license has not yet been selected. Add a `LICENSE` file before accepting contributions or distributing the project. Also review the separate licenses and terms of any datasets, pretrained models, and third-party components used.

---

**DocuShield AI — supporting document review with explainable forensic signals.**
