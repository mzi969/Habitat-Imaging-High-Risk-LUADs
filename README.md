# Habitat imaging-derived “solid component” for predicting high-risk lung adenocarcinomas in part-solid nodules on low-dose CT

**1.Introduction**

This software implements a fully automated habitat imaging pipeline for quantifying “solid component” within part-solid nodules on low-dose computed tomography (LDCT) and predicting high-risk lung adenocarcinoma (LUAD). Using fixed thresholds (density: −447 HU; entropy: 4.201), density and entropy maps are clustered to generate four distinct habitats. The “solid component” is defined as the high density and low entropy region (Label 3), and its volume together with its volume ratio are used as key predictors in a logistic regression model.

**2. File Requirements and Format**

•	CT Images: .nii.gz or .nii (NIfTI format)

•	Segmentation Masks: .nii.gz or .nii (NIfTI format)

**3. Processing Pipeline**

The software executes the following six steps for each case:

**3.1 Step 1: Image Resampling**

•	Description: Both CT and mask images are resampled to isotropic 1 mm³ voxel spacing to standardize spatial resolution.

•	Output: Saved in 1_resampled/ directory.

**3.2 Step 2: Local Entropy Calculation**

•	Description: A 3D local entropy map is computed using a 3×3×3 moving window over the resampled and normalizated CT image.

•	Output: Entropy map saved in 2_entropy/ directory.

**3.3 Step 3: Generation of Four Threshold Masks**

•	Description: Within the original mask region, four binary masks are created by applying the density and entropy thresholds:

o	High Density: CT values ≥ –447 HU

o	Low Density: CT values < –447 HU

o	High Entropy: Entropy values ≥ 4.201

o	Low Entropy: Entropy values < 4.201

•	Output: Four masks saved in 3_threshold_masks/ directory.

**3.4 Step 4: Fusion into a 4 Label Image**

•	Description: The four binary masks are combined pixel wise to create a single 4 label image according to the following rule:

o	Label 1: Low density & Low entropy → Green

o	Label 2: Low density & High entropy → Blue

o	Label 3: High density & Low entropy (“Solid component”) → Red

o	Label 4: High density & High entropy → Yellow

•	Output: Fused label image saved in 4_fusion_masks/ directory.

**3.5 Step 5: Volume Statistics**

•	Description: For each of the four labels, the number of voxels, volume (mm³), and volume ratio (relative to the total mask volume, %) are calculated.

•	Note: Because of prior resampling to 1 mm spacing, each voxel corresponds to 1 mm³.

•	Output: Statistics are displayed in the log and used for the next step.

**3.6 Step 6: High Risk LUAD Prediction**

•	Description: Using the volume of Label 3 (high density & low entropy) in cm³ and its volume ratio in %, together with the patient’s sex, a logistic regression model calculates the probability of high risk LUAD.

•	Formula: logit = −2.808 − 0.697 × sex (man=0, woman=1) + 1.702 × Label3_volume (cm³) + 0.018 × Label3_ratio (%), probability = 1 / (1 + exp(−logit)).

•	Decision Rule:
o	High risk LUAD (Class 1): probability > 0.171
o	Low risk LUAD (Class 0): probability ≤ 0.171

•	Output: Prediction result saved as a CSV file in 5_prediction_results/ directory.

**3.7 Step 7: Visualization Generation**

•	Description: After prediction, a composite figure is automatically generated to visually summarize the analysis. This figure displays:
o	CT map (lung window)
o	Entropy map
o	Habitat labels (with color legend)
o	“solid component” (Label 3) with annotation showing its volume (mm³), ratio (%), probability, and final prediction.

•	Output: Saved as 05_composite.png in the 6_prediction_results/directory.

**4. Disclaimer**

**4.1 Intended Use**

This software at the current stage is primarily intended for research purposes. While it can provide valuable clinical references, it should be integrated with comprehensive clinical evaluation rather than used as the sole basis for clinical decision-making.

**4.2 Limitations**

•	Requires accurate CT images and tumor masks; results are highly dependent on the quality of the input mask.

•	Performance may vary with different patient populations, CT scanner models, and acquisition protocols.

•	The fixed thresholds (–447 HU, 4.201) were derived from a specific study population; they may not be optimal for other cohorts without recalibration.
________________________________________
**5. Continuous Improvement**

We are committed to the ongoing validation and improvement of this tool. Users are encouraged to:

•	Provide feedback on clinical utility and performance.

•	Participate in validation studies.

•	Report any discrepancies with clinical outcomes.

•	Share suggestions for enhancement.

For questions or feedback, please contact the software distributor or the original study authors.


