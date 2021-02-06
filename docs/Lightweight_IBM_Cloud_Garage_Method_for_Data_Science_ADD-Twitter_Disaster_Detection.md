---
title: The Lightweight IBM Cloud Garage Method for Data Science
subtitle: Architectural Decisions Document
author: Tim B. Herbstrith
date: 2021-01-31
---

![IBM Data and Analytics Reference Architecture. © 2020: IBM
Corporation](media/image1.png)

Data Source
===========

Technology Choice
-----------------

Training data is provided in *CSV* format. All data is provided in a
single file, which has the following schema.

    root
     |-- id: integer (nullable = true)
     |-- keyword: string (nullable = true)
     |-- location: string (nullable = true)
     |-- text: string (nullable = true)
     |-- target: integer (nullable = true)

Data is accessed via the official *Kaggle* API. Access credentials are
stored in the object cloud and are retrieved during run-time of the
notebook.

Justification
-------------

-   Using the official API ensures access to the data in accordance to
    *Kaggle's* usage policies. Additionally, data can be accessed in a
    reproducible way and no input data needs to be uploaded.
-   The *CSV* format is chosen, as it is the data format provided by
    *Kaggle*.
-   No realtime data access is needed during the training.

Enterprise Data
===============

Technology Choice
-----------------

No enterprise data is used in the project.

Justification
-------------

Enterprise data is not relevant to the use case.

Streaming analytics
===================

Technology Choice
-----------------

At the moment, there are no plans to integrate streaming technology in
the project. However, it might be interesting to stream new *Twitter*
data for classification in the future.

Justification
-------------

Training and test data is provided as fixed tables by *Kaggle*.

Data Integration
================

Technology Choice
-----------------

Data cleansing and transformation is carried out in *Jupyter* notebooks
running on the *IBM Cloud Pak for Data* service. The environment used
throughout the whole project is defined in a *YAML* file. It sets
environment variables and handles the required packages.

Although, the required data throughput is very small (input file size \<
10 MB), I chose to use *Apache Spark* as a back-end for data processing.

Since the official API of *Kaggle* is chosen, the source system is
standardized and one can assume that changes to input data are minimal
during the lifetime of the project.

Justification
-------------

-   *Jupyter* notebooks were chosen for their ease of use. Additionally,
    they lend themselves to presentations of code.
-   *IBM Cloud Pak for Data* was chosen as a cost effective cloud
    computing platform which is optimized for analytical tasks. Effort
    for set up of the environment is negligible.
-   *Apache PySpark* was chosen for its vertical scalability. As
    mentioned before, this would not be necessary for the data source of
    the project. However, this project is carried out for training
    reasons anyways.

Data Repository
===============

Technology Choice
-----------------

I chose an *IBM Cloud Object Store* service as the data repository for
the project. All data is stored in *Parquet* files.

Justification
-------------

-   Object store technology is very versatile and cost effective.
-   *IBM Cloud Object Store* is very well integrated into *IBM Cloud Pak
    for Data*. Thus, set up of the services and their integration comes
    with minimal effort.
-   Disadvantages regarding data access as compared to *SQL* databases
    can be mitigated by choosing the right serialization formats,
    allowing for indexation and partition.
-   The *Parquet* format allows for partition, indexation and—crucial for
    the *PySpark* framework—can be written in parallel.
-   As the data can be retrieved at any time from source no special
    requirements for fault tolerance and backup strategies were
    identified.

Discovery and Exploration
=========================

Technology Choice
-----------------

Discovery and exploration is carried out in *Jupyter* notebooks. To aid
visualization of tables, `pyspark.sql.DataFrame`-s are transformed into
`pandas.DataFrame`-s (limiting the number of rows before
transformation). Interactive visualizations are created using *Plotly*.
Metrics are calculated using the *Apache Spark* API whenever possible.

Justification
-------------

-   Interactive visualizations greatly augment the data discovery and
    exploration phase as one can zoom in to the data or drop entries
    dynamically.
-   `pandas.DataFrame`-s are rendered beautifully in *Jupyter* notebooks
    by default. After limiting the number of rows to a sensible value,
    the overhead created by transforming the dataframes is negligible.
-   Calculating metrics directly using the *Apache Spark* API leverages
    the native vertical scalability of `pyspark.sql.DataFrame`-s or
    `pyspark.RDD`-s respectively.

Actionable Insights
===================

Technology Choice
-----------------

To gain insights I am using *Apache Spark* and *Apache SparkML*.
Whenever possible, existing models and pre-processors are used.
Convolutional neural networks are created through the *Keras* API of
*Tensorflow*. Unfortunately, I was not able to find a mature API for
training *Keras* models and predicting using *Keras* models leveraging
the vertical scalability of *Apache Spark*.

Justification
-------------

-   *Apache Spark* provides an API similar to *scikit-learn* that allows
    rapid pre-processing, training, and evaluation of machine learning
    models. However, in contrast to *scikit-learn* the *Apache Spark*
    framework is natively able to be parallelized.
-   *Keras* provides a very highlevel and relatively easy to use API for
    defining, training, and evaluating deep learning models. *Tensorflow*
    is the default back-end for *Keras* and supports GPUs natively (if
    configured accordingly).
-   All frameworks allow for efficient serialization and loading of
    models.

Applications / Data Products
============================

Technology Choice
-----------------

As this projects aims to compete in a *Kaggle* data competition, our
data product is the submission of labels for the test data set provided.
The data product must adhere to the specifications stated by *Kaggle*
and will be submitted using the official *Kaggle* API. *Kaggle* will
then evaluate the submission using the *F1*-score.

Justification
-------------

-   A submission to the challenge is the main goal of the project.
-   *Kaggle* dictates format of data products.
-   The official API provides a convenient way to interact with *Kaggle*.

Security, Information Governance, and Systems Management
========================================================

Technology Choice
-----------------

The project relies 100% on cloud PaaS/SaaS. API and application keys are
used for authentification. User access to notebook kernels and object
store are handeld by *IBM*.

Justification
-------------

-   The projects solely relies on public data. Hence, no special
    requirements for data security were identified.
-   All notebooks are published on *GitHub*. Hence, special care has to
    be taken as not to expose private information like API keys. As a
    consequence, API keys are stored as environment variables and are retrieved
    at run-time.
-   Since running kernels and usage of storage can incur costs, user
    access to these resources was chosen very restrictive.
