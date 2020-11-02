# Generate Music based on Emotion

- [Generate Music based on Emotion](#generate-music-based-on-emotion)
    + [Step 0 Preparing dataset](#step-0-preparing-dataset)
    + [Step 1 Extra Feature](#step-1-extra-feature)
    + [Step 2 Find the best CNN/CNN+RNN model](#step-2-find-the-best-cnn-cnn-rnn-model)
      - [CNN model](#cnn-model)
      - [CNN+RNN model](#cnn-rnn-model)
      - [Compare model](#compare-model)
    + [Step 3](#step-3)

### Step 0 Preparing dataset

The dataset is at [DEAM dataset](http://cvml.unige.ch/databases/DEAM/), it includes more than 1800 music data with midi file.

By using the online convert website [bearaudiotool](https://www.bearaudiotool.com/mp3-to-midi) we convert its format into wav file.

We upload the datasets on [Onedrive].

### Step 1 Extra Feature

We should explain the word we will used in the next:
1. **'emotion label'**: a $1\times 2$ vector, include 2 real numbers. One for the value of Arousal and another for the value of Valence.
2. **'data'**: one music song in the dataset, should more than 45 seconds long, with 60 emotion labels.
3. **'fragment'**: one data have 60 half-second fragment. Each emotion label corresponds to one fragment.
4. **'fragment size'**: it depends on the sampling frequency and the number of features exacted in one sampling.

Three things we do in this step:

Firstly, we check and find all data which is available on both format and randomly sample 400 data for training and 100 data for testing. We name them as "400-100 dataset". "Available" means the data have a proper length in piano roll for midi (piano roll is around 4500 long) and more than 45 seconds for wav.

Secondly, we exact features from the data. We create 4 datasets in total, size of **each data** are showed below:

|         | wav                          | midi                      |
| ------- | ---------------------------- | ------------------------- |
| CNN     | 60fragment\*50sf\*128feature | 60frame\*50sf\*128feature |
| CNN+RNN | 60fragment\*1sf\*128feature  | 60frame\*1sf\*128feature  |

The 'sf' means 'sampling frequency'. For CNN, it is 50 and for CNN+RNN it is 1.

In CNN model, we assume fragments in one data are **not related to each other**, so we have $500\times 60=3000$ fragments in total, and now 'fragment' equals 'data' in the usual sense. In CNN+RNN model, we ignore the dimension 'sf' because it equals 1.

Thus we get 4 datasets in the end, 2 of the dataset shape is **3000fragment\*50\*128**, and 2 of the dataset shape is **500data\*60\*128**.

Finally, we save all the datasets.

### Step 2 Find the best CNN/CNN+RNN model

#### CNN model

It is just a huge CNN.

Input shape is 50\*128.

The architect is:
Conv2D(64,3,relu) ->Conv2D(64,3,relu) ->maxpooling(2); then it split intotwo branch for valence and arousal.Conv2D(64,3,relu) ->Conv2D(64,3,relu) ->maxpooling(2)->Conv2D(128,3,relu)->maxpooling(2)  ->dropout(0.25  dropped)  ->Conv2D(256,3,relu)  ->max-pooling(2) ->dropout(0.25) ->Conv2D(256,3,relu) ->maxpooling(2) ->dropout(0.25)->Conv2D(256,3,relu) ->maxpooling(1,3) ->dropout(0.25) ->flatten ->dense(256)->dropout(0.5) ->dense(256) ->dropout(0.5) ->dense(1)

Output shape is 1\*2

#### CNN+RNN model

We do a simple CNN first and then run a RNN(bi-gru).

Input shape is 60\*128.

The architect is:
Conv2D(**cf**,3,relu) ->batchNormalization ->reshape; then it split into two branch for valence and arousal.dense(**vaDense**) ->Bi-GRU(**Gru**) ->Dense(1) ->flatten

Output shape is 60*2.

The cf, vaDense and Gru are the hyper-parameter we will decide by grid search.

#### Compare model

We load all the datasets and run 2 models.

The range for CNN+RNN grid search is:
cf: 8,16
vaDense: 8, 16
Gru: 8,16,32
batch_size: 5,10,15.

And we find the best model is CNN+RNN in cf=16, vaDense=8, Gru = 32, batch_size = 15.

### Step 3













