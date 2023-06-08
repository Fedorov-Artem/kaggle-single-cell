# kaggle-single-cell
Code for kaggle single cell competition (got bronze medal).

## Goal of the Competition
The goal is to predict how DNA, RNA, and protein measurements co-vary in single cells. Cells were isolated from four healthy human donors, and then cultured and incubated for several days at 37ºC. From each culture plate at each sampling time point, cells were collected for measurement. Each cell was measured using one of two technologies: Multiome or CITEseq. The Multiome kit measures chromatin accessibility (DNA) and gene expression (RNA), while the CITEseq kit measures gene expression (RNA) and surface protein levels.

All the data including chromatin accessibility, gene expression and surface protein levels is already pre-processed. At the middle of the competition, organizers also released raw values. For both Multiome and CITEseq tasks, and for both train and test datasets, some metadata is also available, including sampling day and donor ID for each cell.

The task is as follows:
* For the Multiome samples: given chromatin accessibility, predict gene expression.
* For the CITEseq samples: given gene expression, predict protein levels.

For both Multiome and CITEseq the test data consists of either cells from donor unseen in train data, but sampled on the same day with samples from other donors (public test set) or cells sampled from known donors in a day that is not present in the train data (part of private test set), there is also a chunk of test dataset that refers to both unknown donor and sampling day (also part of private test set). Like in other kaggle competitions, the public test set is used to give participants some clue on how well their model works during the competition, as each time a participant submits prediction, kaggle site displays the target metric result calculated for the public part of the test dataset. As for the private part of the test dataset, it is only used at the end of the competition to rank the submissions.

More detailed information is available on [kaggle site](https://www.kaggle.com/competitions/open-problems-multimodal).

## Solution overview
The competition includes two different tasks, each requires a separate model to make predictions.

Multiome inputs include about 220 thousand features that are to be used to predict expression of about 23 thousand genes. Total multiome train dataset is about 90 Gb, compared to size of available RAM of about 16 Gb. But multiome input data is about 97% sparse, this means only about 3% of values are non-zero. This means, it is possible to convert the inputs into sparse format in chunks and fit the sparse inputs into RAM. Another big challenge is the fact, that given resources available for a kaggle notebook, it is not possible to fit a separate model for each of 23 thousand targets. The way to get some predictions despite this sort of restraint is to use dimensionality reduction both for the inputs and for the targets.

CITEseq dataset includes about 22 thousand inputs and 140 targets. It is possible to load all the train inputs into the memory, but does not allow fitting a model with all the 22 thousand inputs. What's more important, it is possible to fit an individual model for each of 140 targets.

Both models use preprocessed in a separate notebook dimensionally reduced data, features prepared from metadata and some pre-selected features directly. For both models, TruncatedSVD was used as a dimensionality reduction method, as this algorithm accepts sparse data as input.

## Notebooks
* main-notebook-cite.ipynb - notebook used to train and cross-validate CITE model
* main-notebook-multiome.ipynb - notebook used to train final multiome model
* prepare-svd-for-cite.ipynb - notebook used for CITE input dimensionality reduction
* prepare-svd-for-multiome.ipynb - notebook used for multiome dimensionality reduction, both for raw and pre-processed inputs (only pre-processed data was used for final submission)

## Story
I've entered the completion about a month after its launch. It didn't take long to notice that out of four donors there were three males and a female, and the donor, who's cells were only present in the test dataset, was male. At the very beginning, I used to leave out data from one of the sampling days or one of the donors while performing cross-validation. In the case of a female donor left out, the cross-validation results were so much worse, than in the case of predicting results for one of the male donors, that I decided to start leaving out either one of the male donors or one of the sampling days. Few days before the end of the competition, when I was running out of kaggle GPU quota, I started using only sampling days for cross-validation, as the private part of the test dataset is entirely from the unknown sampling day.

I tried to find chromatins, corresponding to genes, and genes, corresponding to proteins, and to use that information to improve prediction by adding the corresponding features directly to the inputs.  I found out that it was possible to do just that using the ensembl_rest public and free API. Not for all proteins, corresponding genes were in the available inputs, but adding those that were present directly to the model's inputs in most cases boosted the cross-validation score significantly. But it turned out that the best way to select important genes among 22 thousand available inputs was to add directly to inputs all the available genes chunk by chunk and then select those with high feature importance. Adding features with high feature importance directly to the inputs improved CITEseq's model performance very significantly. As for chromatins, corresponding to genes, I found out that on average one chromatin is about 100 times shorter than an average gene, so in the best case I only had chromatin accessibility for a small part of the actual gene's DNA. Then I've checked the correlation between chromatin accessibility and the corresponding gene expression and found only a few dozens of  cases, where the correlation was higher than 0.1. So, I decided to give up the idea to try to fit any of the 23000 thousand of multiome targets to an individual model, even the simplest possible and to focus on predicting the truncated SVD components of the target.

I've calculated the truncated SVD components from the raw data inputs and tried to add them as features together with SVD components from the pre-processed data. Some of those SVD components from raw data had very high feature importance, but they did not increase the cross-validation results. When I've tried to submit the results, I found out that on the public part of the test dataset, the predictions were actually worse than predictions made without truncated SVD components of raw data.

The EDA notebook (eda-notebook.ipynb) that contains code for experiments I've described earlier, and some data visualizations, is available in this repository.

A few days before the end of the competition, I noticed that some of the 140 CITEseq targets are predicted much better than others, with the best correlation between prediction and ground truth about 0.87 and the worst about 0.1. So, I divided all the targets into three groups and used different parameters for models, predicting targets of each group. Then I used the approach same for models, predicting multiome's target truncatedSVD components - used stronger models to predict first components and added more protection from overfitting to the models predicting later components. This approach helped me to increase the final score significantly.

## Summary and what can be improved
During the competition, my results were relatively low on the leaderboard. But after the competition's end, when private dataset results were published, it turned out that my models generalize better to the private test dataset. I didn't make many submissions, but mostly checked the cross-validation results, and this approach worked fine.

Most participants used artificial neural networks to make predictions - and many of them ended with overfitting solutions. I also wanted to try NN models, but chose to spend this time working with the raw data instead. The very top performers pre-processed the raw data themselves or added some more processing to data, prepared by the organizers. I do not think that searching for  better GBDT model's parameters or putting more effort into feature selection  can significantly improve the results. The best way for further improvement would be to use NN models.
