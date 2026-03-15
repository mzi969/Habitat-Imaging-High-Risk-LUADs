# Habitat Imaging Analysis for High-Risk Lung Adenocarcinoma (LUAD) Prediction on Low-Dose Computed Tomography
**1.Introduction**
This software implements a fully automated habitat imaging pipeline for quantifying solid components within part-solid nodules on low-dose computed tomography (LDCT) and predicting high-risk lung adenocarcinoma (LUAD). Using the Otsu threshold method, density and entropy maps are clustered to generate four distinct habitats. The solid component is defined as the high density & low entropy region, and its volume is used as the key predictor (Definition 3). The algorithm demonstrated superior diagnostic performance (AUC = 0.871 in training, 0.865 in external test) compared to conventional diameter and density measurements.

**2. File Requirements and Format**
•	CT Images: .nii.gz or .nii (NIfTI format)
•	Segmentation Masks: .nii.gz or .nii (NIfTI format)

