## 1. Data Exploration and Preprocess

### 1.1 Exploratory analysis of data

- The [spatial distribution]of the sites.
- [Correlation analysis] data of different sites.
- [Clustering] of different kinds of stations in Beijing.

### 1.2 Data Preprocess

[Data preprocess]then split the dataset into training, val and aggr dataset.

1. Data preprocess

   Steps of data preprocess:

   1. Remove duplicated data. Some of the hour data are duplicated, remove them.
   2. Missing value processing. If hour level data are missing for all stations for 5 hours in a row, all (X,y) data that have these missing data in X or y are droped. Then if data are missing for all stations for less than 5 hours in a row, data before and after missing data are used to generate padding data linearly. In some cases, data for some specific stations are nan, then data from the nearest station will be used to pad.

2. Split the data

   All data points that are valid after data preprocess will be split into 3 parts : training set, validation set and aggregation set. 

   Training set is used for the training of single models, and usually data from 20170101-20180328 will be used in the training set. 

   Validation set will be used for selecting the best single models from the checkpoints of all single models. Then all best single models will be aggregated on the validation set and eveluated finally on the aggregation set. The aggregation model will be used for the final prediction.

### 1.3 [Oversampling]

1. Why oversampling?

 
   [Symmetric mean absolute percentage error (SMAPE)] is used in this competation as evaluation metric. In SMAPE, relative error matters rather than absolut error, as shown in the function.

   However, loss functions like L1 loss, L2 loss and huber loss are applied in different models and they all aim at decreasing absolute error rather than relative error. So if models are trained using original data and these 3 loss functions, trained models would be optimized to fit data points with huge number rather than data points with smaller numbers, which would lead to larger SAMPE when evaluating with validation set and test with test set.

2. Oversamping Strategies

   Training data from 20170101-20180328 are used in the training data. Oversampling steps are as follows:

   1. PM2.5 mean of y is caculated for every (X,y) pair, and all data points in the training set are sorted in ascending order. 
   2. Smallest **oversample_part** of all datapoints are picked and repeated for **repeats** times and are appended to the original training dataset. So (1+**repeats*oversample_part**) times the original amount of training data are finally used to generate training data batch (X,y), which may help to shift the optimization target from those loss functions to SMAPE. 

   **Oversample_part** and **repeats** are hyperparameters which suitable values can be found by random search or grid search. Oversampling lead to a 0.02~0.04 improvement on SMAPE of validation set.

## 2. Models

#### 2.1 seq2seq

[Seq2seq] model is a machine learning model that use **decoder** and **encoder** to learn serialized feature pattern from data. Seq2seq model is applied to a lot of machine learning applications, especially NLP applications like Machine translation. In this project, seq2seq is applied to generate time series forecast of different granularity, which are **Day model** and **Hour model**. The basic graph of seq2seq model is as follows.


1. Day model

   The air condition seem to be very cyclical every day, as shown in the 3rd part in  [bj_aq_data_exploration] and below. So the basic seq2seq model would be **Day model**, which means that we just predict the mean value of all aq parameters in the next 2 days, and then overlay the parameter trend during 24 hours to generate the final prediction.



   The computation graph of Day model is as follws.
   

2. Hour model, Predicting 2 days together

3. Hour model, Predicting 1 day at a time

### 2.2 xgboost

### 2.3 models aggregation


