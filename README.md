# Code Files Instructions

## **Data**

- The original data was obtained from [Mimic-III website](https://physionet.org/content/mimiciii/1.4/), including **ADMISSIONS.csv**, **DIAGNOSES_ICD.csv**, **PROCEDURES_ICD.csv**, and **PATIENTS.csv**.
  
- Tables named **ccs_multi_dx_tool_2015.csv** and **ccs_multi_pr_tool_2015.csv** were downloaded from [this repository](https://github.com/svenhalvorson/icd_ccs_elix/tree/master/Multi_Level_CCS_2015).
  
- The file **older15patients.csv** was created from **ADMISSIONS.csv** and **PATIENTS.csv** using SQL below.

```SQL
CREATE TABLE PATIENTS_ADMISSIONS_CLEANED AS
SELECT 
    A.`SUBJECT_ID`,
    A.`HADM_ID`,
    A.`ADMITTIME`,
    A.`INSURANCE`,
    A.`RELIGION`, 
    A.`MARITAL_STATUS`,
    A.`ETHNICITY`,
    A.`DIAGNOSIS`,
    P.`GENDER`,
    P.`DOB`
FROM
    admissions_cleaned` A
LEFT JOIN
    `PATIENTS` P ON P.`SUBJECT_ID` = A.`SUBJECT_ID`;
```

```SQL
SELECT *, YEAR(`ADMITTIME`) - YEAR(`DOB`) AS age
FROM PATIENTS_ADMISSIONS_CLEANED
WHERE YEAR(`ADMITTIME`) - YEAR(`DOB`) > 15 AND YEAR(`ADMITTIME`) - YEAR(`DOB`) < 200
AND SUBJECT_ID NOT IN (
    SELECT SUBJECT_ID
    FROM PATIENTS_ADMISSIONS
    GROUP BY SUBJECT_ID
    HAVING COUNT(*) = 1
)
ORDER BY SUBJECT_ID;
```

  
- Output files from our notebooks include: **admissions_cleaned_15.csv**, **diagnoses_label.csv**, **procedures_label.csv**, **diagnose.dates**, **diagnose.pids**, **diagnose.seqs**, **diagnose.types**, **procedures.dates**, **procedures.pids**, **procedures.seqs**, and **procedures.types**.

## **Code Files**

### **Data Cleaning and Preprocessing.ipynb**

The initial part of this file involves data cleaning and the code for converting ICD-9 encoding to CCS encoding. The subsequent part demonstrates how to transform patients' diagnoses in each admission into a large matrix, as mentioned in section 3.2.3 Batch processing of our report：

### **model_RNN.ipynb**

The first section covers padding sequences and one-hot vector transformation, along with data splitting techniques. The second part encompasses all the models experimented with during the experience.

- **MLP model**: We experimented with one-layer, two-layer, and three-layer MLP models, varying the dense neurons individually in 128, 256, 512, 1024, and 2048. Here, we include only the best-performing model in our file.
  
- **Feed Forward GRU and Feed Forward LSTM**: These are basic GRU/LSTM models with a TimeDistributed(Dense) layer. Neurons within the GRU/LSTM layer were tested with sizes 128, 256, 512, 1024, and 2048 individually.
  
- **Bidirectional GRU and Bidirectional LSTM**: We applied Bidirectional() technique to GRU/LSTM layers and explored models with two and three RNN layers.
  
- **Bidirectional + LReLU + Self Attention**: After identifying the best model from previous steps, we further improved it by trying alternative activation functions like ReLU, Sigmoid, and LeakyReLU. Additionally, we incorporated a self-attention layer, significantly enhancing prediction results.
  
- **Top-30 recall plot**: Since we previously used only 10 epochs, we ran 20 epochs and plotted performance to demonstrate convergence. It also compares the performance between our improved model with and without the self-attention layer.

## **LIG-Doctor**: 

The code is sourced from [this repository](https://github.com/jfrjunio/LIG-Doctor/tree/master), with modifications made by our team.

**Running LIG-Doctor:**

```python
# Testing the installation
python
import theano #should execute smoothly

# After all is set
# Model Training - Builds model to be saved in a file called model.npz:
python ./LIG-Doctor.py ./LIG ./model --maxConsecutiveNonImprovements 5 --hiddenDimSize 135 --batchSize 32 --nEpochs 5 --LregularizationAlpha 0.001 --dropoutRate 0.3

# Model Testing:
python ./LIG-Doctor-test.py ./LIG ./model.npz --hiddenDimSize 135 --batchSize 32





