# kaggle-single-cell
Code for kaggle single cell competition (got bronze medal).
## Goal of the Competition
The goal is to predict how DNA, RNA, and protein measurements co-vary in single cells. Cells were isolated from four healthy human donors, and then cultured and incubated for several days at 37ºC. From each culture plate at each sampling time point, cells were collected for measurement. Each cell was measured using one of two technologies: Multiome or CITEseq. The Multiome kit measures chromatin accessibility (DNA) and gene expression (RNA), while the CITEseq kit measures gene expression (RNA) and surface protein levels.

The task is as follows:
* For the Multiome samples: given chromatin accessibility, predict gene expression.
* For the CITEseq samples: given gene expression, predict protein levels.

All the data including chromatin accessibility, gene expression and surface protein levels was given in a pre-processed form. At the middle of the competition, organizers also released raw values for the same data.

More detailed information is available on [kaggle site](https://www.kaggle.com/competitions/open-problems-multimodal).
## Solution overview
The competition includes two different tasks, each with its own dataset and each requires a separate model to make predictions. So, the solution would also consist of two independent models.

Multiome inputs include about 220 thousand features that are to be used to predict expression of about 23 thousand genes. Total multiome train dataset is about 90 Gb, compared to size of available RAM about 16 Gb. But multiome input data is about 97% sparse, this means only about 3% of values are non-zero. Another big challenge is the fact, that given resources available for a kaggle notebook, it is not possible to fit a separate model for each of 23 thousand targets. The way to get some predictions despite this sort of restraints is to use dimensionality reduction both for inputs and for targets to get some predictions.

CITE task includes about 22 thousand inputs and 140 targets. It is possible to load all the train dataset inputs into the memory, but it is not possible to fit a model using all the 22 thousand inputs. What's more important, it is possible to fit an individual model for each of 140 targets. As for input data, I will use both dimensionality reduced data and some pre-selected features directly.

For both models, TruncatedSVD was used a dimensionality reduction method
## Notebooks
* main-notebook-cite.ipynb - notebook used to train and cross-validate CITE model
* main-notebook-multiome.ipynb - notebook used to train final multiome model
* prepare-svd-for-cite.ipynb - notebook used for CITE input dimensionality reduction
* prepare-svd-for-multiome.ipynb - notebook used for multiome dimensionality reduction, both for raw and pre-processed inputs (only pre-processed data was used for final submission)
