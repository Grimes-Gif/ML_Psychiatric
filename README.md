![MLinfo](https://user-images.githubusercontent.com/90532657/174424871-785ce7d2-3e6d-46a6-b7aa-ce24235c9a5f.PNG)


# Psychiatric classification
Machine learning project for CS4641 at Georgia Institute of Technology
Collab: Saif Syed Ali, Ibraheem Abutair, Matthew Johnson, Micah Grimes


## Introduction
The objective of this project is to use machine learning to gain insight onto potential predictors or biomarkers present in the brain of patients diagnosed with psychiatric disorders. 

Ideally the inclusion criteria is as follows: 
1. subject must have been diagnosed with a psychiatric disorder by a medical professional
2. Must be having a current episode of major depression OR has had an episode of major depression in the past 6 months. 

We believe that machine learning is especially useful in this field because it allows for the standardization, comparison, and subsequent analysis of datasets that come from different metrics/studies. Succesful completion of this project would allow us to provide physicians and researchers with possible predictive measures for the development of psychiatric disorders. This would allow more conclusive 

 


![image](https://user-images.githubusercontent.com/8241982/182699684-c2b5c357-c85d-4445-9da6-af2226580717.png)
Pictured above is a generic electrode map, with the top representing the front of the head. The electrodes are named after their respective brain region: for example, the “O1” and “O2” electrodes monitor brain activity in the occipital lobe. Likewise, the “FP1” and “FP2” electrodes monitor brain activity in the frontal cortex. 



## Methods

### Data types
Our first task was to go about trying to decide what measurements would be most useful to building a psychiatric classifier. Naturally neuro imaging and EEG data came to mind as they are both non-invasive and risk free. However, in our search we found that securing MRI and fMRI images was extremely difficult with many studies being blocked behind a paywalls, not providing enough scope to be a psychiatric classifier, or providing a sparse amount of data points, most of these issues likely being a result of the high expenses that come with operating and owning neuroimaging technology. Due to these limitations, we setteled for using EEG datasets, which is much more inexpensive to perform and use. 


### Dataset
The dataset we selected was an EEG dataset with 2 sets of labels from kaggle. While this data was much more plentiful than other sets we came across, it unfortunately had very little documentation, so data cleaning and analysis was a bit harder than usual.

#### Dimensionality and Description
1. Rows = 945 patients
2. Columns = 1149, for a total of 1148 recorded features (One column being an empty divider between the recordings)
3. The first 114 columns were the frequencies of each sensor with respect to a particular frequency range, categorically known as brain waves
  - 19 electrodes, 6 brain wave classes = 114 readings 
4. The 115th column is empty serving as the aformentioned divider
5. The rest of the dataset from column 116 to 1149 were coherence measurments, which capture the 'communication' between regions by measuring the frequency in which the two regions share wave formations. This is more formally known as phase-amplitude coupling. 
6. The first set of labels consisted of strings that were the specific disorder each patient had. This set consisted of 12 unique values. 
7. The second set of labels consisted of the more general groupings of each disorder, called the main disorders. This set consisted of 8 unique values.

### Data cleaning, Feature extraction, Engineering, and Reduction
Initially, PCA was run on the entire dataset in order to reduce dimensionality. This was a must, as the initial dataset contains over 1000 features. After running PCA, we found that aiming for a 70% recovered variance led to 40+ components, which still wasn't ideal. Thus, we decided to divide the data into subsets using feature selection. 
First, columns 112 - 1000 were all coherence values between each electrode. Since we were not particularly interested in these values for our current goals, we dropped these columns. This instantly made our dataset much easier to work with. 
Next, for the supervised learning portion, we focused on major depressive disorder (MDD), which allowed us to drop all rows pertaining to other disorders. Lastly, we focused on the frontal parietal lobe (FP1 and FP2 electrodes), as this a critical brain region involved with higher-order thinking. 
All of these feature selections allowed us to create a workable dataset, which we run several learning algorithms on. 

SUPERVISED LEARNING:
Due to the incredibly large number of features (1000+) in the dataset, it is important to conduct informed data analysis to perform targeted machine learning. 
This was accomplished by specifically examining the FP1 and FP2 (frontal parietal cortex) electrodes of depressed patients and healthy controls. The figures below show the frequency correlations of FP1/FP2 between depressed (blue) and control (red) for Alpha, Beta, Delta, and Theta waves. This was done to determine which waves had the largest differences between the two groups. 

While the Alpha, Beta, and Theta waves seem to have considerable overlap, the Delta waves tell a different story: the control subjects seem to have lower frequencies of delta waves, while the depressed subjects seem to have higher frequencies. This is illustrated by the apparent skew of blue dots towards the upper right corner and the skew of red dots towards the bottom left corner in the Delta Wave figure. 

![image](https://user-images.githubusercontent.com/8241982/182698982-f50c5d82-7a0e-4111-8fbc-051dc4e03af5.png)

This hypothesis is furthered by the results of running a Kernel Density Estimation on the same electrode data.  As seen below, the frequency of Delta waves is skewed to the right when compared to the control, corroborating the evidence of the scatter plots. 


 

This apparent difference immediately tells us that the delta waves are worth investigating further. Thus, we took our machine learning approach to build a predictive model that analyzes the delta waves in the FP1 and FP2 electrodes.

SUPERVISED MACHINE LEARNING

In the medical field, Support Vector Machines (SVMs) are a popular method of supervised machine learning. This is due to the fact SVMs specialize in binary classification, resulting in a robust medical tool to use for predictive diagnosis. 

The results of applying an SVM to the above dataset is shown below. We first used a linear kernel, but the results yielded a poor ~52% accuracy rate on the test data. This makes sense, given the abysmal boundaries of classification yielded by SVC w/ linear kernel and LinearSVC. Applying an RBF Kernel yielded even worse results, as the decision boundaries were nearly non-existent. However, when a polynomial kernel was used, the accuracy of the model increased by an enormous 15%, averaging at around 71% accuracy. The bottom right figure illustrates why this was the case; the decision boundaries are much more nuanced and capture the pattern of the control and depression data.   

![image](https://user-images.githubusercontent.com/8241982/182699034-5cfb6186-a088-46ba-aa37-c21f56f372b2.png)

Other Supervised ML Algorithms used were:

Stochastic Gradient Descent (SGD): 71.4% Accuracy
Multilayer Neural Network (MPL): 58%
Decision Tree: 62%

The poor results given by the MLP are likely due to the lack of training data – there were only around 400 training points for the network to learn from. If more training data (several thousands) was present, the network would likely have performed better. 
The decision tree did not perform well due to the number of features in the dataset, as the algorithm usually breaks down once a large number of features is reached. 

Interestingly, the SGD algorithm, on average, performed the best out of all algorithms. 





Feature engineering involved normalizing the wave frequencies in order to yield more accurate/consistent models.

Data preprocessing consisted largely of isolating subdata sets and particular features to work with that would increase metrics for our models, this was because the main bulk of the work, transferring time series data to frequency domain had already been done by the authors of the dataset using FFT. Additional preprocessing done by the dataset authors included removing EOG (eye movemnt) artifacts from the frontal electrodes, allowing for a high confidence of signal to be neural in nature. 







### ML workflow
We currently plan on mainly using K-means clustering (unsupervised) to analyze the data and potential patterns that may emerge. We will also employ linear regression (supervised) in order to draw relationships between variables and draw predictions based on said relationships.

## Results
We expect to find certain features in specific brain regions that are consistent across most study subjects. A metric of success would be a high level of correlation between one or more physical brain anomolies and a corresponding psychiatric diagnosis. For example, a measure of success could be a high degree of correlation between grey matter volume in the caudate nucleus and recurrance of MDD. 

Discussion
Using machine learning, we hope to be able to advance the understanding of the physical biomarkers associated with major depressive disorder. Using a database of images, we are able to distinguish unique similarities between patients. 
MRI imaging makes it possible to use imaging as a data type, however there are some issues with MRI imaging only holding greyscale information as well as some images maybe cloudy. We would also need to corrected positioning/perspective differences in images.
Not much research has been done in this field using machine learning. We hope to advance this method in that field and find very useful results. 
Information on biomarkers for MDD can be very beneficial to doctors and researchers, but most of all to the patients suffering from the condition.
The only risk associated with this work would be a waste of time and resources, which is well worth the risk for the oppurtunity this offers.
The run times for the algorithms will vary based on the algorithm created but should not be longer than a day for each runtime and a couple of weeks until ready for deployment.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------








MIDTERM REPORT:

Introduction
The objective of this project is to use machine learning to gain insight on physical biomarkers present in the brain of patients diagnosed with major depressive disorder
(MDD) and other similar psychiartic disorders. Our inclusion criteria is as follows: subject must have been diagnosed with MDD and/or other psychiatric disorders by a 
medical professional and must be having a current mental episode OR in the past 6 months. 

The current research in this field is plentiful, but very few have used machine learning on imaging data in order to generate patterns. We believe that machine learning
is especially useful in this field because it allows for the standardization, comparison, and subsequent analysis of datasets that come from different metrics/studies. 
Succesful completion of this project would allow us to provide physicians and researchers with possible predictive measures for the development of MDD and similar disorders.
This would allow physicians to mark certain individuals as "high-risk" for physiomental health issues and can provide preventative care to the patient.

Methods
The dataset type used will be numerical data taken from the Department of Psychiatry of Boramae Medical Center. The nature of the sets is processed EEGs signals from 
individuals with psychiatric disorders. The entire data sheet contains EEG results from 945 subjects, which will be more than enough to run machine learning algorithms on. 
Each table from the dataset has 400 features. The targets will be discrete, as we hope to use these targets to determine specific biomarkers of psychiatric disorders.
Since the data was already standardized by the researchers (quality MRI studies will use a standardized template on which to "map" the brain of the subject onto), 
standardization should not be an issue. We will attempt to perform dimensionality reduction via a covariance filter. 
We currently plan on mainly using K-means clustering (unsupervised) to analyze the data and potential patterns that may emerge. We will also employ linear regression
(supervised) in order to draw relationships between variables and draw predictions based on said relationships.

Results - Updated

We have employed a "trial and error" approach to our selection of datasets. While we started out with a focus on fMRI data, we quickly found that fMRI/MRI 
datasets are hard to come by. Most datasets do not deal directly with major depressive disorder (MDD), and the ones that do are unusable for a variety of 
reasons, ranging from lack of data to a lack of normalization between brain scans/brain regions. Additionlly, we found it exceedingly difficult
and out of scope to attempt to write a machine learning algorithm that utilizes computer vision to analyze brain scans. We explored a variety of more 'psychiatric'
data sets that involved rating scales for MDD symptoms alongside demographic data. While in some data sets, we were able to find clusters/corellations, 
it was all known info that could already be discerned with statistical analysis. Similar data sets we examined that did not already possess published
findings yieled no clusters/corellations. After concluding we could not work with psyciatric sets, we decided to shift our focus to numeral MRI datasets. 
With this dataset, we ran into a different set of issues. Namely, normalization appeared to be an issue again - comparing MRI data from different studies and 
different brain regions would run the risk of faulty or inconclusive results, due to inconsistent metrics of data collection, ranging from MRI machine tonnage 
to individual differences in brain structure.

We finally ended up looking at an EEG dataset. EEG data is usually easier to work with than MRI data, as EEG data is less visual and more numerical. 
Additionally, the brain waves recorded from EEG. Early results from PCA are promising, but we plan to experiment with a variety of methods. 

EEG data report: Due to the extremely large dimensionality of the data set, measuring nearly 1000 features, finding a good way to visualize our results has become a point of tension. PCA while 
useful, still generates around 40 to 60 principle componenets on the range of 90 to 95 percent of the variance. Clustering has been performed but analysis is still underway.

