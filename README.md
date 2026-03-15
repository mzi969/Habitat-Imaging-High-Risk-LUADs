# Habitat-Imaging-High-Risk-LUADs
Software Manual: Habitat Imaging Analysis for High-Risk Lung Adenocarcinoma (LUAD) Prediction on Low-Dose Computed Tomography

1. Software Overview
1.1 Software Introduction
This software implements a fully automated habitat imaging pipeline for quantifying solid components within part-solid nodules on low-dose computed tomography (LDCT) and predicting high-risk lung adenocarcinoma (LUAD). Using the Otsu threshold method, density and entropy maps are clustered to generate four distinct habitats. The solid component is defined as the high density & low entropy region, and its volume is used as the key predictor (Definition 3). The algorithm demonstrated superior diagnostic performance (AUC = 0.871 in training, 0.865 in external test) compared to conventional diameter and density measurements.
1.2 Key Features
•	Single Case Processing: Analyze CT images and segmentation masks for individual patients.
•	Batch Processing: Automatically process multiple cases with automatic file pairing.
•	Habitat Imaging Pipeline: Six-step workflow (resample → entropy maps → threshold masks → fusion masks → volume statistics → prediction) based on Otsu-derived thresholds from the training cohort.
•	Interpretable Visualization: Outputs a 4 label image mapping distinct habitats.
•	Result Export: Export prediction results and habitat volumes to CSV and Excel formats.
1.3 Software Interface
The software features a tabbed graphical user interface with four main sections:
Tab	Description
📁 Input & Processing	File selection, mode choice (single/batch), parameter input, processing controls, and real time log display
📊 Single Case Results	Detailed results of the last processed single case, including volumes of four habitat labels and final prediction
📈 Batch Results	Summary table of all processed cases in batch mode, with status and prediction for each case
📋 About	Software version, description, processing steps, citation information, and prediction threshold
________________________________________
2. File Requirements and Format
2.1 Supported File Formats
•	CT Images: .nii.gz or .nii (NIfTI format)
•	Segmentation Masks: .nii.gz or .nii (NIfTI format)
2.2 File Naming Conventions
The software automatically matches CT and mask files. The required naming pattern is:
•	CT file: casename.nii.gz
•	Mask file: casename_mask.nii.gz
Example:
CT File	Mask File
patient001.nii.gz	patient001_mask.nii.gz
subject_123.nii	subject_123_mask.nii
If files follow this convention, they will be correctly paired during batch scanning.
________________________________________
3. Software Operation Guide
3.1 Launching the Software
Double click the executable Habitat_LUAD.exe. The main window will appear.
3.2 Processing Modes
3.2.1 Single Case Processing
1.	Select "Single Case" radio button.
2.	Click "Browse CT" to select the CT file.
3.	Click "Browse Mask" to select the corresponding mask file.
4.	Configure output directory (default: C:\Users\YourName\HabitatLUAD_Output).
5.	(Optional) Adjust thresholds: Density Threshold (–405 HU) and Entropy Threshold (4.204) are fixed based on the study; modification is not recommended.
6.	Click "Start Processing". The log will show progress; upon completion, the Single Case Results tab opens automatically.
3.2.2 Batch Processing
1.	Select "Batch Processing" radio button.
2.	Click "Browse Directory" to select the folder containing all CT and mask files.
3.	Click "Scan for File Pairs" – the software will list all detected valid pairs (green check marks).
4.	Review the detected file pairs in the list.
5.	Configure output directory and thresholds as needed.
6.	Click "Start Processing". The progress bar updates for each case; results appear in the Batch Results tab in real time.
3.3 Common Settings
•	Output Directory: Location where results for each case will be saved (subfolders are created automatically).
•	Density Threshold (HU): Default −405 HU (Otsu-derived, 95% CI: −414.5 to −395.2).
•	Entropy Threshold: Default 4.204 (Otsu-derived, 95% CI: 4.188 to 4.222).
•	Prediction Threshold (Label 2 volume): 172 mm³ (optimal cutoff from the training set). This threshold is used to classify high risk LUAD.
________________________________________
4. Processing Pipeline
The software executes the following six steps for each case:
4.1 Step 1: Image Resampling
•	Description: Both CT and mask images are resampled to isotropic 1 mm³ voxel spacing to standardize spatial resolution.
•	Interpolation: Linear for CT, nearest neighbor for mask.
•	Output: Saved in 1_resampled/ directory.
4.2 Step 2: Local Entropy Calculation
•	Description: A 3D local entropy map is computed using a 3×3×3 moving window over the resampled CT image.
•	Formula: Shannon entropy based on a 32 bin histogram of voxel intensities within the window.
•	Output: Entropy map saved in 2_entropy/ directory.
4.3 Step 3: Generation of Four Threshold Masks
•	Description: Within the original mask region, four binary masks are created by applying the density and entropy thresholds:
o	High Density: CT values ≥ –405 HU
o	Low Density: CT values < –405 HU
o	High Entropy: Entropy values ≥ 4.204
o	Low Entropy: Entropy values < 4.204
•	Output: Four masks saved in 3_threshold_masks/ directory.
4.4 Step 4: Fusion into a 4 Label Image
•	Description: The four binary masks are combined pixel wise to create a single 4 label image according to the following rule:
o	Label 1: high density & high entropy
o	Label 2: high density & low entropy
o	Label 3: low density & high entropy
o	Label 4: low density & low entropy
•	Output: Fused label image saved in 4_fusion_masks/ directory.
4.5 Step 5: Volume Statistics
•	Description: For each of the four labels, the number of voxels, volume (mm³), and volume ratio (relative to the total mask volume) are calculated.
•	Note: Because of prior resampling to 1 mm spacing, each voxel corresponds to 1 mm³.
•	Output: Statistics are displayed in the log and used for the next step.
4.6 Step 6: High Risk LUAD Prediction
•	Decision Rule: If the volume of Label 2 (high density & low entropy) exceeds the threshold (172 mm³), the case is classified as high risk LUAD (Class 1); otherwise low risk LUAD (Class 0).
•	Output: Prediction result saved as a CSV file in 5_prediction_results/ directory.
________________________________________
5. Output Structure
For each processed case, a subfolder named after the case is created in the output directory, containing the following files:
Habitat LUAD_Output/
└── CaseName/
    ├── 1_resampled/
    │   ├── CaseName_resample.nii.gz          (Resampled CT, 1mm isotropic)
    │   └── CaseName_resample_mask.nii.gz     (Resampled mask, aligned)
    ├── 2_entropy/
    │   └── CaseName_resample_entropy.nii.gz  (3D entropy map)
    ├── 3_threshold_masks/
    │   ├── CaseName_high_density_mask.nii.gz
    │   ├── CaseName_low_density_mask.nii.gz
    │   ├── CaseName_high_entropy_mask.nii.gz
    │   └── CaseName_low_entropy_mask.nii.gz
    ├── 4_fusion_masks/
    │   └── CaseName_fused_labels.nii.gz      (4 label image: labels 1–4)
    └── 5_prediction_results/
        └── CaseName_prediction.csv          (Prediction result and label volumes)
5.1 Output File Descriptions
Directory	File Pattern	Description
1_resampled/	*_resample.nii.gz	Resampled CT image
	*_resample_mask.nii.gz	Resampled mask aligned with CT
2_entropy/	*_resample_entropy.nii.gz	Local entropy map
3_threshold_masks/	*_high_density_mask.nii.gz	Binary mask: CT ≥ –405 HU
	*_low_density_mask.nii.gz	Binary mask: CT < –405 HU
	*_high_entropy_mask.nii.gz	Binary mask: entropy ≥ 4.204
	*_low_entropy_mask.nii.gz	Binary mask: entropy < 4.204
4_fusion_masks/	*_fused_labels.nii.gz	4 label image
5_prediction_results/	*_prediction.csv	CSV file containing case name, volumes of four labels, and prediction (0/1)
________________________________________
6. Results Interpretation
6.1 Single Case Results Tab
After single case processing, the Single Case Results tab displays a table with the following columns:
Column	Description	Format / Example
Case	Patient identifier (derived from filename)	patient001
Label1 Volume	Volume of high density & high entropy region (mm³)	125.34
Label2 Volume	Volume of high density & low entropy region (mm³)	214.56
Label3 Volume	Volume of low density & high entropy region (mm³)	89.12
Label4 Volume	Volume of low density & low entropy region (mm³)	45.78
Prediction	Final classification	High risk / Low risk
•	The Prediction cell is color coded: red for high risk, green for low risk.
6.2 Batch Results Tab
Column	Description	Format / Example
Case	Patient identifier (derived from filename)	patient001
Label1 Volume	Volume of high density & high entropy region (mm³)	125.34
Label2 Volume	Volume of high density & low entropy region (mm³)	214.56
Label3 Volume	Volume of low density & high entropy region (mm³)	89.12
Label4 Volume	Volume of low density & low entropy region (mm³)	45.78
Prediction	Final classification	High risk / Low risk
Status	Processing status (Success / Error message)	Success
•	Prediction is color coded similarly.
•	Failed cases appear with a light red background and an error message in the Status column.
6.3 Prediction Threshold
•	Cutoff value: 172 mm³ (volume of Label 2 – high density & low entropy)
•	High risk LUAD (Class 1): Label 2 volume ≥ 172 mm³
•	Low risk LUAD (Class 0): Label 2 volume < 172 mm³
This threshold is derived from the original study (definition 3) and demonstrated superior diagnostic performance in the external test set (AUC = 0.865).
________________________________________
7. Disclaimer
7.1 Intended Use
This software at the current stage is primarily intended for research purposes. While it can provide valuable clinical references, it should be integrated with comprehensive clinical evaluation rather than used as the sole basis for clinical decision-making.
7.2 Limitations
•	Requires accurate CT images and tumor masks; results are highly dependent on the quality of the input mask.
•	Performance may vary with different patient populations, CT scanner models, and acquisition protocols.
•	The fixed thresholds (–405 HU, 4.204, 172 mm³) were derived from a specific study population; they may not be optimal for other cohorts without recalibration.
________________________________________
8. Continuous Improvement
We are committed to the ongoing validation and improvement of this tool. Users are encouraged to:
•	Provide feedback on clinical utility and performance.
•	Participate in validation studies.
•	Report any discrepancies with clinical outcomes.
•	Share suggestions for enhancement.
For questions or feedback, please contact the software distributor or the original study authors.
________________________________________
