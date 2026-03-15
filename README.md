# Habitat Imaging Analysis for High-Risk Lung Adenocarcinoma (LUAD) Prediction on Low-Dose Computed Tomography
**1.Introduction**
This software implements a fully automated habitat imaging pipeline for quantifying solid components within part-solid nodules on low-dose computed tomography (LDCT) and predicting high-risk lung adenocarcinoma (LUAD). Using the Otsu threshold method, density and entropy maps are clustered to generate four distinct habitats. The solid component is defined as the high density & low entropy region, and its volume is used as the key predictor (Definition 3). The algorithm demonstrated superior diagnostic performance (AUC = 0.871 in training, 0.865 in external test) compared to conventional diameter and density measurements.

**2. File Requirements and Format**

•	CT Images: .nii.gz or .nii (NIfTI format)

•	Segmentation Masks: .nii.gz or .nii (NIfTI format)

**3. Processing Pipeline**
The software executes the following six steps for each case:

**3.1 Step 1: Image Resampling**

•	Description: Both CT and mask images are resampled to isotropic 1 mm³ voxel spacing to standardize spatial resolution.

•	Interpolation: Linear for CT, nearest neighbor for mask.

•	Output: Saved in 1_resampled/ directory.

**3.2 Step 2: Local Entropy Calculation**

•	Description: A 3D local entropy map is computed using a 3×3×3 moving window over the resampled CT image.

•	Formula: Shannon entropy based on a 32 bin histogram of voxel intensities within the window.

•	Output: Entropy map saved in 2_entropy/ directory.

**3.3 Step 3: Generation of Four Threshold Masks**

•	Description: Within the original mask region, four binary masks are created by applying the density and entropy thresholds:

o	High Density: CT values ≥ –405 HU

o	Low Density: CT values < –405 HU

o	High Entropy: Entropy values ≥ 4.204

o	Low Entropy: Entropy values < 4.204

•	Output: Four masks saved in 3_threshold_masks/ directory.

**3.4 Step 4: Fusion into a 4 Label Image**

•	Description: The four binary masks are combined pixel wise to create a single 4 label image according to the following rule:

o	Label 1: high density & high entropy

o	Label 2: high density & low entropy

o	Label 3: low density & high entropy

o	Label 4: low density & low entropy

•	Output: Fused label image saved in 4_fusion_masks/ directory.

**3.5 Step 5: Volume Statistics**

•	Description: For each of the four labels, the number of voxels, volume (mm³), and volume ratio (relative to the total mask volume) are calculated.

•	Note: Because of prior resampling to 1 mm spacing, each voxel corresponds to 1 mm³.

•	Output: Statistics are displayed in the log and used for the next step.

**3.6 Step 6: High Risk LUAD Prediction**

•	Decision Rule: If the volume of Label 2 (high density & low entropy) exceeds the threshold (172 mm³), the case is classified as high risk LUAD (Class 1); otherwise low risk LUAD (Class 0).

•	Output: Prediction result saved as a CSV file in 5_prediction_results/ directory.
