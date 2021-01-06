---
layout: post
title: "Summary: Opportunities in Machine Learning for Healthcare"
author: Lennart Grosser
date: 2020-10-25
category: "paper-summary"
description: "A summary of the paper Opportunities in Machine Learning for Healthcare, Ghassemi et al., 2018. The paper paper describes the need for machine learning applications in healthcare, gives an overview of clinical data types as well as technical challenges. Further, it describes present and novel clinical opportunities for machine learning applications in healthcare settings. Some sections are left out."
tags: ["machine learning", "healthcare", "clinical data"]
---

> This post is a summary of the paper [Opportunities in Machine Learning for Healthcare, Ghassemi et al., 2018](https://arxiv.org/abs/1806.00388). The paper paper describes the need for machine learning applications in healthcare, gives an overview of clinical data types as well as technical challenges. Further, it describes present and novel clinical opportunities for machine learning applications in healthcare settings. Some sections are left out.

## Why Machine Learning for Healthcare matters
The usage of data to address health problems is crucial: humans collect information about the patient and his condition to determine an appropriate treatment which directly yields the need for complete documentation and precise information extraction.

Modern electronic health records provide more and more data to support clinical staff and allow for more insight about a patient's condition.

Recent machine learning research has shown to be an effective way to leverage the potential of collected patient documentation to create meaningful applications that help both doctors and patients:
* [Development and Validation of a Deep Learning Algorithm for Detection of Diabetic Retinopathy in Retinal Fundus Photographs, Gulshan et al. 2016][1]
* [Deep Learning Algorithms for Detection of Lymph Node Metastases From Breast Cancer, Golden et al., 2017][2]
* [Learning probabilistic phenotypes from heterogeneous EHR data, Pivovarov et al. 2015][3]

## Clinical Data
Data forms the basis for the successful application of most machine learning algorithms. This paper focuses on electronic health records (EHR), an electronic way of documenting the patient history. The EHR can contain many different types of data, such as high-frequency data, vital sign checks that are done every few hours, laboratory test results and written notes documenting the condition or treatments.

The authors differentiate between three types of data:

1. High-frequency monitors real-time data at a patient's bedside: high-frequently sampled monitor signals that needs to be preprocessed before being used for learning or feature extraction.

2. Vital and labs measure biomarkers of a patient's state: Selective measurements of the patient's state, e.g. vital signs or laboratory tests. The time of a measurement may also be a relevant factor.

3. Written notes: textual information that captures any interaction between patient and healthcare staff, e.g. surgery reports or discharge letter. NLP tools may be used to extract information from the documents.

## Challenges in Healthcare ML Tasks

The authors name three technical challenges that should be considered when working on healthcare machine learning applications.

1. Understanding Causality: A fundamental problem in healthcare is to answer causality questions, e.g. _what happens if a doctor administers a treatment?_. Answering such questions can be challenging as the available data is collected observationally and may have been influenced by unknown factors. A short example to visualize the underlying problem: researchers found that asthmatic patients who were admitted to the hospital for pneumonia were more aggressively treated for the infection, thus lowering the subpopulation mortality rate. A learning algorithm may now find that asthma is protective, as it leads to more intense care, which is obviously wrong and may to catastrophic outcomes if such a model would be released and determine the level of care a patient needs. The additional information about how intense the care of a patient was, would have been an important to such a model as it would enable the model to understand the severity of a disease.
2. Missing data: samples missing features are a common problem in machine learning. Three different classifications of missing data are popular: a) missing completely at random (MCAR), b) missing at random (MAR) and c) missing not at random (MNAR). The first (MCAR) describes a fixed probability of a feature missing. In this case, samples that do miss the feature are dropped, so that unbiased learning is possible (complete case analysis). MAR describes the probability of missingness as random conditional on the observed variables, e.g. nurses are less likely to measure lactate levels in patients with traumatic injury while traumatic injuries are documented. If nurses are less likely to measure lactate levels in patients who they believe have low levels, then the lactate measures are MNAR (the measurement of the signal is meaningful). Summarized: information may be conveyed by the absence of a measurement that should not be ignored.
3. Outcomes should be defined carefully: Multiple data sources should be combined to create reliable labels for  a machine learning model. A single data source may be incomplete and yield incorrect outcomes (the EHR indicates a patient had pneumonia but that could also mean he was only _screened_ for pneumonia). Further, a careful comprehension of available labels is necessary: a model should not predict the behavior it has seen in the training data if the training data itself contains mistakes (the model should not aim to predict the behavior of a doctor that is wrong).

## Machine Learning in Healthcare Opportunities

Now what are some of the most promising opportunities to apply machine learning to healthcare?

### Clinical Task automation during diagnosis and treatment
Many tasks performed by clinicians are well-defined and require a small amount of domain adaption and investment. Such tasks can be done by clinicians with a high degree of accuracy, however may take a long time and burden the staff, e.g. encoding textual information about diagnosis and treatment to prepare health insurance accounting.
1. Automating medical image evaluation: Machine learning has shown to be effective for image classification or object detection. Those are tasks that usually require a clinician to consider medical images and possibly conclude a necessary treatment based on the observation. Learning algorithms could process more data in a shorter period of time than a human and highlight only relevant detections, e.g. an image where cancer was detected may be proposed to a doctor for further inspection while other images can be discarded.
2. Automating routine processes: Routine tasks that need to be done for every patient and that are time-consuming, e.g. summarizing a patient's treatment that could be accelerated using NLP methods.

### Clinical Support and Augmentation
1. Standardizing clinical processes: To cope for different amounts of clinical training and experience, treatment choices could be supported by ML predictions, e.g. when a clinician is unsure about which medication sets or doses are most appropriate for a patient, a machine learning model could recommend sets or dosages based on the past condition and treatment while suggesting default dosages to avoid dangerous dosing.
2. Integrating fragmented records: a long patient history may lead to many fragmented pieces of information about the patient that would be required to be scanned manually by a clinician to find useful indications. Automatic review and summary of the patient history could speed up this process.

### Expanding Clinical Capacities
1. Moving towards continuous behavioral monitoring: Non-invasive monitoring through wearables may be a useful way to detect anomalies in the patient's condition faster and more efficient than invasive methods that require a clinician. The evaluation of such high-frequency measurements is almost impossible for a human, however easily possible for an appropriate algorithm.
2. Precision medicine for early individualized treatment: precision medicine describes individual treatment paths for patients. Suggestion of non-standard treatment based on the patient history and similar cases.

## Opportunities for New Research in Machine Learning
The authors further describe future opportunities for machine learning research in healthcare research. I will shortly summarize the key points.

### Data
For most existing work, machine learning models were trained on the largest data set possible and assumed to fit for deployment. This assumption may not hold for clinical settings for two reasons:
1. Internal validity (shift over time): The practices, standard processes or medical definitions in a hospital may change over time. Noisy data containing deprecated information can be a source of error when fed into a machine learning model.
2. External Validity (shift over sources): There is no reason to believe _a priori_ that a machine learning model learned from one hospital will generalize to a new one. Even within one hospital transitions from one medical record system to another one or different patient populations can create non-obvious feature mapping problems.

### Interpretability
Models that are deployed should be able to explain their predictions. Clinical staff cannot be expected to understand clinical, legal and technical issues that come with a black box machine learning model.
Further, future research should strive towards justification instead of interpretability, i.e. a model should be able to defend a particular prediction path. Justification would increase the transparency of the model and its decision process while relieving clinical staff from defending technical behavior they might not understand.

[1]: https://jamanetwork.com/journals/jama/fullarticle/2588763
[2]: https://jamanetwork.com/journals/jama/article-abstract/2665757
[3]: https://www.sciencedirect.com/science/article/pii/S1532046415002233#f0015
