# Expert-level detection of pathologies from unannotated chest X-ray images via self-supervised learning

<details>
<summary>
  <b>Expert-level detection of pathologies from unannotated chest X-ray images via self-supervised learning</b>, Nat. Biomed. Eng (2022). 
  <a href="https://www.nature.com/articles/s41551-022-00936-9" target="blank">[Paper]</a>
	<br><em><a href="https://www.linkedin.com/in/ekin-tiu-0aa467200/">Ekin Tiu</a>, <a href="https://www.linkedin.com/in/ellie-talius/">Ellie Talius</a>, <a href="https://www.linkedin.com/in/pujanpatel24/">Pujan Patel</a>, <a href="https://med.stanford.edu/profiles/curtis-langlotz">Curtis P. Langlotz</a>, <a href="https://www.andrewng.org/">Andrew Y. Ng</a>, <a href="https://pranavrajpurkar.squarespace.com/">Pranav Rajpurkar</a></em></br>
</summary>

```bash
Tiu, E., Talius, E., Patel, P. et al. Expert-level detection of pathologies from unannotated chest X-ray images via self-supervised learning. Nat. Biomed. Eng (2022). https://doi.org/10.1038/s41551-022-00936-9
```
</details>

<img width="848" alt="Screen Shot 2022-09-15 at 10 57 16 AM" src="https://user-images.githubusercontent.com/12751529/190451160-a919b363-6005-4cd4-9633-b194392bd728.png">

This repository contains code to train a self-supervised learning model on chest X-ray images that lack explicit annotations and evalute this model's performance on pathology-classification tasks.

<details>
  <summary>
	  <b>Main Findings</b>
  </summary>

1. **Automatically detecting pathologies in chest x-rays without explicit annotations:** Our method learns directly from the combination of images and unstructured radiology reports, thereby avoiding time-consuming labeling efforts. Our deep learning method is capable of predicting multiple pathologies and differential diagnoses that it had not explicitly seen during training. 
2. **Matching radiologist performance on different tasks on an external test set:** Our method performed on par with human performance when evaluated on an external validation set (CheXpert) of chest x-ray images labeled for the presence of 14 different conditions by multiple radiologists.
3. **Outperforming approaches that train on explicitly labeled data on an external test set:**  Using no labels, we outperformed a fully supervised approach (100% of labels) on 3 out of the 8 selected pathologies on a dataset (PadChest) collected in a different country. We further demonstrated high performance (AUC > 0.9) on 14 findings and at least 0.700 on 53 findings out of 107 radiographic findings that the method had not seen during training.
</details>


## Dependencies
To clone all files:

```git clone https://github.com/rajpurkarlab/CheXzero.git```

To install Python dependencies:

```pip install -r requirements.txt```

## Data 
### Training Dataset
1. Navigate to [MIMIC-CXR Database](https://physionet.org/content/mimic-cxr/2.0.0/) to download the training dataset. Note: in order to gain access to the data, you must be a credentialed user as defined on [PhysioNet](https://physionet.org/settings/credentialing/). 
2. Copy the dataset into the `data/` directory.
3. Run `python preprocess_train_data.py`
4. This should preprocess the chest x-ray images into a Hierarchical Data Format (HDF) format used for training stored at `data/cxr.h5` and extract the impressions section as text from the corresponding chest x-ray radiology report stored at `data/mimic_impressions.csv` .

### Evaluation Dataset

#### CheXpert Dataset
The CheXpert dataset consists of chest radiographic examinations from Stanford Hospital, performed between October 2002
and July 2017 in both inpatient and outpatient centers. Population-level characteristics are unavailable for the CheXpert test
dataset, as they are used for official evaluation on the CheXpert leaderboard. 

The main data (CheXpert data) supporting the results of this study are available at https://aimi.stanford.edu/chexpert-chest-x-rays.

The CheXpert **test** dataset used for official evaluation is hidden from the public to maintain the integrity of the CheXpert competition. 

#### PadChest Dataset
The PadChest dataset contains chest X-rays that were interpreted by 18 radiologists at the Hospital Universitario de San Juan,
Alicante, Spain, from January 2009 to December 2017. The dataset contains 109,931 image studies and 168,861 images.
PadChest also contains 206,222 study reports.

The [PadChest](https://arxiv.org/abs/1901.07441) is publicly available at https://bimcv.cipf.es/bimcv-projects/padchest. Those who would like to use PadChest for experimentation should request access to PadChest at the [link](https://bimcv.cipf.es/bimcv-projects/padchest). 

### Model Checkpoints
Model checkpoints of CheXzero pre-trained on MIMIC-CXR are publicly available at the following [link](https://drive.google.com/drive/folders/19YH2EALQTbkKXdJmKm3iaK8yPi9s1xc-?usp=sharing). Download files and save them in the `./checkpoints/chexzero_weights` directory.

## Running Training
Run the following command to perform CheXzero pretraining. 
```bash
python run_train.py --cxr_filepath "./data/cxr.h5" --txt_filepath "data/mimic_impressions.csv"
```

### Arguments
* `--cxr_filepath` Directory to load chest x-ray image data from.
* `--txt_filepath` Directory to load radiology report impressions text from.

Use `-h` flag to see all optional arguments. 

## Zero-Shot Inference
See the following [notebook](https://github.com/rajpurkarlab/CheXzero/blob/main/notebooks/zero_shot.ipynb) for an example of how to use CheXzero to perform zero-shot inference on a chest x-ray dataset. The example shows how to output predictions from the model ensemble and evaluate performance of the model if ground truth labels are available.

```python
import zero_shot

# computes predictions for a set of images stored as a np array of probabilities for each pathology
predictions, y_pred_avg = zero_shot.ensemble_models(
    model_paths=model_paths, 
    cxr_filepath=cxr_filepath, 
    cxr_labels=cxr_labels, 
    cxr_pair_template=cxr_pair_template, 
    cache_dir=cache_dir,
)
```
In order to use CheXzero for zero-shot inference, ensure the following requirements are met: 
* All input *`images`* must be stored in a single `.h5` (Hierarchical Data Format). See the [`write_h5(cxr_paths)`](https://github.com/rajpurkarlab/internal-chexzero/blob/cleanversion/preprocess_padchest.py#L155) function in [preprocess_padchest.py](https://github.com/rajpurkarlab/internal-chexzero/blob/cleanversion/preprocess_padchest.py) for an example of how to convert a list of paths to `.png` files into a valid `.h5` file. 
* The *ground truth `labels`* must be in a `.csv` dataframe where rows represent each image sample, and each column represents the binary labels for a particular pathology on each sample.
* Ensure all [model checkpoints](https://drive.google.com/drive/folders/19YH2EALQTbkKXdJmKm3iaK8yPi9s1xc-?usp=sharing) are stored in `checkpoints/chexzero_weights/`, or the `model_dir` that is specified in the notebook.

