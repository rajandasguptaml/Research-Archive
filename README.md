# research

Code, data and notebooks behind my papers and ML projects, collected from nine separate repositories into one.
Each folder keeps its original commit history (run `git log -- <folder>`). Where a folder had its own README,
it is still there and describes that project in detail.

Research focus: trustworthy and explainable machine learning for clinical decision support, retrieval-augmented
clinical reasoning, multimodal learning, and evaluation of language models in low-resource languages.
More at [rajandasguptaml.github.io](https://rajandasguptaml.github.io/) and the
[publications page](https://rajandasguptaml.github.io/publications.html).

## Projects

| Folder | What it is | Paper / venue | Year |
|---|---|---|---|
| [hi-reaso](hi-reaso/) | HI-REASO: Hindi numerical-reasoning benchmark for LLMs. Full 3,255-question pool across six tasks, plus two 1,000-question evaluation splits. | [Findings of EMNLP 2026](https://aclanthology.org/2026.findings-emnlp.481/) | 2026 |
| [patient-no-show-uplift](patient-no-show-uplift/) | From Prediction Toward Action: explainable no-show risk modeling and exploratory uplift analysis for appointment management. | see folder README and CITATION.cff | 2026 |
| [neonatal-death-sao-paulo](neonatal-death-sao-paulo/) | Deep learning and classical ML models (KNN, SVC, logistic regression, random forest, XGBoost) to predict neonatal death from Sao Paulo birth records. | [IJPHS](https://doi.org/10.11591/ijphs.v13i1.22577) | 2022 |
| [brain-tumor-cnn](brain-tumor-cnn/) | Brain tumor detection in MRI with Faster R-CNN, plus MNIST CNN baselines and the course report. Computer vision coursework. | course project | 2021 |
| [ml-competitions-practice](ml-competitions-practice/) | HackerEarth competition notebooks (A Fine Windy Day, Slashing Prices, Pet Adoption) and practice notebooks from Chollet and Geron. | - | 2021 |
| [early-work/face-detection-opencv](early-work/face-detection-opencv/) | Haar-cascade face detection on images and video. | - | 2021 |
| [early-work/vqgan-clip-art](early-work/vqgan-clip-art/) | VQGAN+CLIP text-to-image notebook. | - | 2022 |
| [road-accident-thesis](road-accident-thesis/) | Undergraduate thesis: reducing road accidents with an OpenCV-based intelligent system. Abstract only; code to be added. | B.Sc. thesis, AIUB | 2021 |

Related work kept in its own repository: [BRAINS](https://github.com/rajandasguptaml/BRAINS), a retrieval-augmented
system for Alzheimer's detection and monitoring (ICMLA 2025, first author).

## Data files

Large or third-party data files were removed from the history to keep this repository small. Each affected folder
has a DATA.md that lists the removed files and where to obtain them.

## License

Everything in this repository is released under the MIT License (see [LICENSE](LICENSE)) unless a folder contains
its own license file, in which case that file applies to that folder (patient-no-show-uplift has one). Each folder
with notebooks has a requirements.txt listing the packages its code imports.
