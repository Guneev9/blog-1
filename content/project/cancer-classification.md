+++
# Date this page was created.
date = "2018-09-20"

# Project title.
title = "Cancer Classification via Gene Expression"

# Project summary to display on homepage.
summary = "Study and comparison of algorithms for cancer classification using gene expression data."

# Optional image to display on homepage (relative to `static/img/` folder).
image_preview = "Most_variant_features-2.png"

# Tags: can be used for filtering projects.
# Example: `tags = ["machine-learning", "deep-learning"]`
tags = ["machine-learning", "feature-selection", "python", "cancer-detection"]

# Optional external URL for project (replaces project detail page).
#external_link = ""

# Does the project detail page use math formatting?
math = true

# Optional featured image (relative to `static/img/` folder).
#[header]
#image = "Most_variant_features-2.png"
#caption = "Features with high variance showing distinction between cancer and healthy"

+++

## **Overview**

Cancer classification is one of the vital areas of medical sciences. In this project, we examine the Gene Expression data for various types of cancer and classify each cancer into its subtypes. We gather the samples from GEO datasets and select the best features using the class-overlap score, standard deviation by mean ratio, ANOVA – F value and $\chi^2$ (Chi-square)  statistics. We then run the following classifiers: K Nearest Neighbors, Naïve Bayes and Decision Trees. We then checked the accuracy of a classifier against a feature selection method for all combinations and observe a feature selection – classifier pair that gives a better accuracy for a particular dataset.

## **Background**

Early discovery of cancer is an important step in determining the treatment of a patient. Gene expression data can be used to predict the presence of cancer. Traditional way of cancer classification uses the physical attributes and the symptoms of the patient to gain a deeper understanding of the patient’s condition. Because causation and spread of cancer is more related to the genes, it is equally important to study the gene expression data.

## **Goal**
 
Our goal was to use Gene Expression Omnibus (GEO) datasets to build models that can predict the presence of cancer and identify the severity of the cancer. These datasets are freely and publicly available from the [NCBI](https://www.ncbi.nlm.nih.gov/gds). Each dataset has the gene expression data for various samples, with rows of a matrix file denoting the probe id and the columns denoting the samples. We took five GEO datasets, which have been briefly explained below :

### [GSE19804 (Lung Cancer Dataset)](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE19804):
GSE19804 is a lung cancer dataset having 120 samples, with 60 lung cancer tissue samples and 60 normal tissue samples, with 54, 675 probe sets. 

### [GSE27562 (Breast Cancer Dataset)](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE27562):
GSE27562 is a breast cancer dataset having 31 normal samples, 57 malignant breast cancer samples, and 37 benign breast cancer samples, with 54, 675 probe sets. There are 15 gastrointestinal tumor and and 7 brain tumor samples, which we have excluded from the datasets as a part of the experiments. We have also excluded 15 breast cancer samples following surgery, since there is no information about them having shown any signs of cancer after surgery. 

### [GSE32474 (NCI60 Dataset)](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE32474):
GSE32474 dataset contains 9 types of different tumors. It has 18 samples for leukemia, 15 samples for breast cancer, 21 samples for ovarian cancer, 21 samples for colon cancer, 26 samples for melanoma, 18 samples for central nervous system, 23 samples for renal cancer and 26 samples for non-small lung cancer, with 54, 675 probe sets. We have excluded 6 prostate samples, since it is not enough for classification. 

### [GSE33315 (Leukemia Dataset)](http://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE33315):
GSE33315 dataset contains 7 types of leukemia cancer. It has 115 samples of type hyperdiploid, 40 samples of TCF3-PBX1, 99 samples of ETV_RUNX1, 30 samples of MLL, 23 samples of PH , 23 samples of hypodiploid and 83 samples of T-ALL, with 22,283 probe sets. We have excluded 4 CD34 and 4 CD10CD19 samples, since that is too less for classification. We have also excluded 153 samples having no known karyotype to focus on leukemia. 

### [GSE59856 (Pancreatic and Biliary Tract Cancer)](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE59856):
GSE59856 dataset contains 100 pancreatic cancer samples, 98 biliary tract samples, 50 colon cancer samples, 50 stomach cancer samples, 50 esophagus cancer samples, 52 liver cancer samples, 150 healthy samples, with 2555 probe sets.

## **Challenges**

There were two major challenges in this project:

1. High number of features
2. Low number of samples

These two challenges go side-by-side in this scenario. Probe ids of a series metrix file essentialy denote the genes whose expression value has been recorded. As such, they are the features of the datasets. In case of Lung Cancer Dataset (GSE19804), we had as many as 54,675 probe ids. On the other hand, there are only 120 samples in this dataset. This is a very low number of observations as compared to the number of features; a ratio of less than 0.002.
Lung Cancer, Breast Cancer : > 54,000 features, < 150 samples
Pancreatic and Biliary Tract Cancer : 2555 features, 450 samples

## **Actions**

Tried variety of methods for feature selection and model depending on the data:
High Variance features distinguish cancer and normal in case of Lung Cancer
Doesn’t work for others
Tried different feature selection methods including class overlap score, Chi-square, ANOVA-F
Tried different machine learning models including KNN, Naive Bayes, CART

## **Results**

Fairly good results with around 100 features
All results are using stratified repeated random sub-sampling validation
Each cell has (F1 score, number of features)
