

```python
# -*- coding: utf-8 -*-
# Authors: Chitta Ranjan <cran2367@gmail.com>
#
# License: BSD 3 clause
```


```python
import numpy as np
import pandas as pd
from itertools import chain
import warnings

########
from sklearn.preprocessing import LabelEncoder
import tensorflow as tf
from keras.datasets import imdb
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense
from tensorflow.keras.layers import LSTM
from tensorflow.keras.layers import Dropout, Activation, Flatten
from tensorflow.keras.layers import Embedding
from tensorflow.keras.preprocessing import sequence
np.random.seed(7) # fix random seed for reproducibility
from sklearn.model_selection import train_test_split, KFold, StratifiedKFold
import sklearn.metrics
import time

from sklearn.decomposition import PCA
from sklearn.cluster import KMeans

import matplotlib.pyplot as plt
%matplotlib inline

from sgt import Sgt
```


```python

tf.__version__
```




    '2.0.0'



## Test Examples


```python
sgt = Sgt()
```


```python
sequence = np.array(["B","B","A","C","A","C","A","A","B","A"])
alphabets = ["A", "B", "C"]
lengthsensitive = True
kappa = 5
```


```python
sgt.getpositions(sequence = sequence, alphabets = alphabets)
```




    [('A', (array([2, 4, 6, 7, 9]),)),
     ('B', (array([0, 1, 8]),)),
     ('C', (array([3, 5]),))]




```python
sgt.fit(sequence, alphabets, lengthsensitive, kappa, flatten=False)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>A</th>
      <th>B</th>
      <th>C</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>A</th>
      <td>0.369361</td>
      <td>0.442463</td>
      <td>0.537637</td>
    </tr>
    <tr>
      <th>B</th>
      <td>0.414884</td>
      <td>0.468038</td>
      <td>0.162774</td>
    </tr>
    <tr>
      <th>C</th>
      <td>0.454136</td>
      <td>0.068693</td>
      <td>0.214492</td>
    </tr>
  </tbody>
</table>
</div>




```python
corpus = [["B","B","A","C","A","C","A","A","B","A"], ["C", "Z", "Z", "Z", "D"]]
```


```python
s = sgt.fit_transform(corpus)
print(s)
```

    [[0.90616284 1.31002279 2.6184865  0.         0.         0.86569371
      1.23042262 0.52543984 0.         0.         1.37141609 0.28262508
      1.35335283 0.         0.         0.         0.         0.
      0.         0.         0.         0.         0.         0.
      0.        ]
     [0.         0.         0.         0.         0.         0.
      0.         0.         0.         0.         0.         0.
      0.         0.09157819 0.92166965 0.         0.         0.
      0.         0.         0.         0.         0.         0.92166965
      1.45182361]]



```python
sequence_test = [['a', 'b'], ['a', 'b', 'c'], ['e', 'f']]
```


```python
sequence_model_test = Sgt(kappa=10, lengthsensitive=True)
```


```python
result_test = sequence_model_test.fit_transform(corpus=sequence_test)
```


```python
result_test
```




    array([[0.        , 0.39428342, 0.        , 0.        , 0.        ,
            0.        , 0.        , 0.        , 0.        , 0.        ,
            0.        , 0.        , 0.        , 0.        , 0.        ,
            0.        , 0.        , 0.        , 0.        , 0.        ,
            0.        , 0.        , 0.        , 0.        , 0.        ],
           [0.        , 0.41059877, 0.15105085, 0.        , 0.        ,
            0.        , 0.        , 0.41059877, 0.        , 0.        ,
            0.        , 0.        , 0.        , 0.        , 0.        ,
            0.        , 0.        , 0.        , 0.        , 0.        ,
            0.        , 0.        , 0.        , 0.        , 0.        ],
           [0.        , 0.        , 0.        , 0.        , 0.        ,
            0.        , 0.        , 0.        , 0.        , 0.        ,
            0.        , 0.        , 0.        , 0.        , 0.        ,
            0.        , 0.        , 0.        , 0.        , 0.39428342,
            0.        , 0.        , 0.        , 0.        , 0.        ]])




```python
sequence_model_test.alphabets
```




    ['a', 'b', 'c', 'e', 'f']



## Protein Sequence Data Analysis

The data used here is taken from www.uniprot.org. This is a public database for proteins. The data contains the protein sequences and their functions. In the following, we will demonstrate 
- clustering of the sequences.
- classification of the sequences with the functions as labels.


```python
protein_data=pd.DataFrame.from_csv('../data/protein_classification.csv')
X=protein_data['Sequence']
def split(word): 
    return [char for char in word] 

sequences = [split(x) for x in X]
print(sequences[0])
```

    ['M', 'E', 'I', 'E', 'K', 'T', 'N', 'R', 'M', 'N', 'A', 'L', 'F', 'E', 'F', 'Y', 'A', 'A', 'L', 'L', 'T', 'D', 'K', 'Q', 'M', 'N', 'Y', 'I', 'E', 'L', 'Y', 'Y', 'A', 'D', 'D', 'Y', 'S', 'L', 'A', 'E', 'I', 'A', 'E', 'E', 'F', 'G', 'V', 'S', 'R', 'Q', 'A', 'V', 'Y', 'D', 'N', 'I', 'K', 'R', 'T', 'E', 'K', 'I', 'L', 'E', 'D', 'Y', 'E', 'M', 'K', 'L', 'H', 'M', 'Y', 'S', 'D', 'Y', 'I', 'V', 'R', 'S', 'Q', 'I', 'F', 'D', 'Q', 'I', 'L', 'E', 'R', 'Y', 'P', 'K', 'D', 'D', 'F', 'L', 'Q', 'E', 'Q', 'I', 'E', 'I', 'L', 'T', 'S', 'I', 'D', 'N', 'R', 'E']


### Generating sequence embeddings


```python
sgt = Sgt(kappa = 1, lengthsensitive = False)
```


```python
embedding = sgt.fit_transform(corpus=sequences)
```


```python
embedding.shape
```




    (2112, 400)



#### Sequence Clustering
We perform PCA on the sequence embeddings and then do kmeans clustering.


```python
pca = PCA(n_components=2)
pca.fit(embedding)
X=pca.transform(embedding)

print(np.sum(pca.explained_variance_ratio_))
df = pd.DataFrame(data=X, columns=['x1', 'x2'])
df.head()
```

    0.6432744907364925





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>x1</th>
      <th>x2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.384913</td>
      <td>-0.269873</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.022764</td>
      <td>0.135995</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.177792</td>
      <td>-0.172454</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.168074</td>
      <td>-0.147334</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.383616</td>
      <td>-0.271163</td>
    </tr>
  </tbody>
</table>
</div>




```python
kmeans = KMeans(n_clusters=3, max_iter =300)
kmeans.fit(df)

labels = kmeans.predict(df)
centroids = kmeans.cluster_centers_

fig = plt.figure(figsize=(5, 5))
colmap = {1: 'r', 2: 'g', 3: 'b'}
colors = list(map(lambda x: colmap[x+1], labels))
plt.scatter(df['x1'], df['x2'], color=colors, alpha=0.5, edgecolor=colors)
```




    <matplotlib.collections.PathCollection at 0x147c494e0>




![png](output_23_1.png)


#### Sequence Classification
We perform PCA on the sequence embeddings and then do kmeans clustering.


```python
y = protein_data['Function [CC]']
encoder = LabelEncoder()
encoder.fit(y)
encoded_y = encoder.transform(y)
```

We will perform a 10-fold cross-validation to measure the performance of the classification model.


```python
kfold = 10
X = pd.DataFrame(embedding)
y = encoded_y

random_state = 1

test_F1 = np.zeros(kfold)
skf = KFold(n_splits = kfold, shuffle = True, random_state = random_state)
k = 0
epochs = 50
batch_size = 128

for train_index, test_index in skf.split(X, y):
    X_train, X_test = X.iloc[train_index], X.iloc[test_index]
    y_train, y_test = y[train_index], y[test_index]
    X_train = X_train.as_matrix(columns = None)
    X_test = X_test.as_matrix(columns = None)
    
    model = Sequential()
    model.add(Dense(64, input_shape = (X_train.shape[1],))) 
    model.add(Activation('relu'))
    model.add(Dropout(0.5))
    model.add(Dense(32))
    model.add(Activation('relu'))
    model.add(Dropout(0.5))
    model.add(Dense(1))
    model.add(Activation('sigmoid'))
    model.compile(loss='binary_crossentropy', optimizer='adam', metrics=['accuracy'])
    
    model.fit(X_train, y_train ,batch_size=batch_size, epochs=epochs, verbose=0)
    
    y_pred = model.predict_proba(X_test).round().astype(int)
    y_train_pred = model.predict_proba(X_train).round().astype(int)

    test_F1[k] = sklearn.metrics.f1_score(y_test, y_pred)
    k+=1
    
print ('Average f1 score', np.mean(test_F1))
```

    Average f1 score 1.0


## Weblog Data Analysis
This data sample is taken from https://www.ll.mit.edu/r-d/datasets/1998-darpa-intrusion-detection-evaluation-dataset. 
This is a network intrusion data containing audit logs and any attack as a positive label. Since, network intrusion is a rare event, the data is unbalanced. Here we will,
- build a sequence classification model to predict a network intrusion.

Each sequence contains in the data is a series of activity, for example, {login, password}. The _alphabets_ in the input data sequences are already encoded into integers. The original sequences data file is also present in the `/data` directory.


```python
darpa_data = pd.DataFrame.from_csv('../data/darpa_data.csv')
darpa_data.columns
```




    Index(['seqlen', 'seq', 'class'], dtype='object')




```python
X = darpa_data['seq']
sequences = [x.split('~') for x in X]
```


```python
y = darpa_data['class']
encoder = LabelEncoder()
encoder.fit(y)
y = encoder.transform(y)
```

### Generating sequence embeddings
In this data, the sequence embeddings should be length-sensitive. The lengths are important here because sequences with similar patterns but different lengths can have different labels. Consider a simple example of two sessions: `{login, pswd, login, pswd,...}` and `{login, pswd,...(repeated several times)..., login, pswd}`. While the first session can be a regular user mistyping the password once, the other session is possibly an attack to guess the password. Thus, the sequence lengths are as important as the patterns.


```python
sgt_darpa = Sgt(kappa = 5, lengthsensitive = True)
```


```python
embedding = sgt_darpa.fit_transform(corpus=sequences)
```


```python
pd.DataFrame(embedding).to_csv(path_or_buf='tmp.csv', index=False)
pd.DataFrame(embedding).head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>2391</th>
      <th>2392</th>
      <th>2393</th>
      <th>2394</th>
      <th>2395</th>
      <th>2396</th>
      <th>2397</th>
      <th>2398</th>
      <th>2399</th>
      <th>2400</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.069114</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.000000</td>
      <td>0.0</td>
      <td>4.804190e-09</td>
      <td>7.041516e-10</td>
      <td>0.0</td>
      <td>2.004958e-12</td>
      <td>0.000132</td>
      <td>1.046458e-07</td>
      <td>5.863092e-16</td>
      <td>7.568986e-23</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.540296</td>
      <td>5.739230e-32</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.785666</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>1.950089e-03</td>
      <td>2.239981e-04</td>
      <td>2.343180e-07</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.528133</td>
      <td>1.576703e-09</td>
      <td>0.0</td>
      <td>2.516644e-29</td>
      <td>1.484843e-57</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 2401 columns</p>
</div>



#### Applying PCA on the embeddings
The embeddings are sparse. We, therefore, apply PCA on the embeddings.


```python
from sklearn.decomposition import PCA
pca = PCA(n_components=35)
pca.fit(embedding)
X = pca.transform(embedding)
print(np.sum(pca.explained_variance_ratio_))
```

    0.9887812984792304


#### Building a Multi-Layer Perceptron Classifier
The PCA transforms of the embeddings are used directly as inputs to an MLP classifier.


```python
kfold = 3
random_state = 11

test_F1 = np.zeros(kfold)
time_k = np.zeros(kfold)
skf = StratifiedKFold(n_splits=kfold, shuffle=True, random_state=random_state)
k = 0
epochs = 300
batch_size = 15

# class_weight = {0 : 1., 1: 1.,}  # The weights can be changed and made inversely proportional to the class size to improve the accuracy.
class_weight = {0 : 0.12, 1: 0.88,}

for train_index, test_index in skf.split(X, y):
    X_train, X_test = X[train_index], X[test_index]
    y_train, y_test = y[train_index], y[test_index]
    
    model = Sequential()
    model.add(Dense(128, input_shape=(X_train.shape[1],))) 
    model.add(Activation('relu'))
    model.add(Dropout(0.5))
    model.add(Dense(1))
    model.add(Activation('sigmoid'))
    model.summary()
    model.compile(loss='binary_crossentropy', optimizer='adam', metrics=['accuracy'])
    
    start_time = time.time()
    model.fit(X_train, y_train ,batch_size=batch_size, epochs=epochs, verbose=1, class_weight=class_weight)
    end_time = time.time()
    time_k[k] = end_time-start_time

    y_pred = model.predict_proba(X_test).round().astype(int)
    y_train_pred = model.predict_proba(X_train).round().astype(int)
    test_F1[k] = sklearn.metrics.f1_score(y_test, y_pred)
    k += 1
```

    Model: "sequential_12"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    dense_30 (Dense)             (None, 128)               4608      
    _________________________________________________________________
    activation_30 (Activation)   (None, 128)               0         
    _________________________________________________________________
    dropout_20 (Dropout)         (None, 128)               0         
    _________________________________________________________________
    dense_31 (Dense)             (None, 1)                 129       
    _________________________________________________________________
    activation_31 (Activation)   (None, 1)                 0         
    =================================================================
    Total params: 4,737
    Trainable params: 4,737
    Non-trainable params: 0
    _________________________________________________________________
    Train on 73 samples
    Epoch 1/300
    73/73 [==============================] - 1s 9ms/sample - loss: 0.1489 - accuracy: 0.4658
    Epoch 2/300
    73/73 [==============================] - 0s 138us/sample - loss: 0.1350 - accuracy: 0.5890
    Epoch 3/300
    73/73 [==============================] - 0s 138us/sample - loss: 0.1403 - accuracy: 0.5205
    Epoch 4/300
    73/73 [==============================] - 0s 142us/sample - loss: 0.1272 - accuracy: 0.6849
    Epoch 5/300
    73/73 [==============================] - 0s 126us/sample - loss: 0.1189 - accuracy: 0.7945
    Epoch 6/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.1198 - accuracy: 0.7260
    Epoch 7/300
    73/73 [==============================] - 0s 155us/sample - loss: 0.1100 - accuracy: 0.8904
    Epoch 8/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.1015 - accuracy: 0.8767
    Epoch 9/300
    73/73 [==============================] - 0s 146us/sample - loss: 0.0999 - accuracy: 0.8767
    Epoch 10/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.1011 - accuracy: 0.8356
    Epoch 11/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0967 - accuracy: 0.9178
    Epoch 12/300
    73/73 [==============================] - 0s 140us/sample - loss: 0.0816 - accuracy: 0.9178
    Epoch 13/300
    73/73 [==============================] - 0s 151us/sample - loss: 0.0858 - accuracy: 0.9041
    Epoch 14/300
    73/73 [==============================] - 0s 132us/sample - loss: 0.0762 - accuracy: 0.8904
    Epoch 15/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0826 - accuracy: 0.8904
    Epoch 16/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0757 - accuracy: 0.9178
    Epoch 17/300
    73/73 [==============================] - 0s 137us/sample - loss: 0.0740 - accuracy: 0.9041
    Epoch 18/300
    73/73 [==============================] - 0s 132us/sample - loss: 0.0781 - accuracy: 0.9041
    Epoch 19/300
    73/73 [==============================] - 0s 137us/sample - loss: 0.0696 - accuracy: 0.9178
    Epoch 20/300
    73/73 [==============================] - 0s 145us/sample - loss: 0.0615 - accuracy: 0.9041
    Epoch 21/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0620 - accuracy: 0.9178
    Epoch 22/300
    73/73 [==============================] - 0s 152us/sample - loss: 0.0618 - accuracy: 0.9041
    Epoch 23/300
    73/73 [==============================] - 0s 143us/sample - loss: 0.0684 - accuracy: 0.9041
    Epoch 24/300
    73/73 [==============================] - 0s 132us/sample - loss: 0.0614 - accuracy: 0.9178
    Epoch 25/300
    73/73 [==============================] - 0s 138us/sample - loss: 0.0594 - accuracy: 0.9041
    Epoch 26/300
    73/73 [==============================] - 0s 151us/sample - loss: 0.0577 - accuracy: 0.9041
    Epoch 27/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0629 - accuracy: 0.9041
    Epoch 28/300
    73/73 [==============================] - 0s 137us/sample - loss: 0.0488 - accuracy: 0.9178
    Epoch 29/300
    73/73 [==============================] - 0s 143us/sample - loss: 0.0541 - accuracy: 0.9178
    Epoch 30/300
    73/73 [==============================] - 0s 142us/sample - loss: 0.0586 - accuracy: 0.9178
    Epoch 31/300
    73/73 [==============================] - 0s 152us/sample - loss: 0.0521 - accuracy: 0.9041
    Epoch 32/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0524 - accuracy: 0.9178
    Epoch 33/300
    73/73 [==============================] - 0s 138us/sample - loss: 0.0519 - accuracy: 0.9178
    Epoch 34/300
    73/73 [==============================] - 0s 143us/sample - loss: 0.0490 - accuracy: 0.9178
    Epoch 35/300
    73/73 [==============================] - 0s 139us/sample - loss: 0.0414 - accuracy: 0.9178
    Epoch 36/300
    73/73 [==============================] - 0s 155us/sample - loss: 0.0447 - accuracy: 0.9041
    Epoch 37/300
    73/73 [==============================] - 0s 152us/sample - loss: 0.0413 - accuracy: 0.9178
    Epoch 38/300
    73/73 [==============================] - 0s 154us/sample - loss: 0.0470 - accuracy: 0.9178
    Epoch 39/300
    73/73 [==============================] - 0s 161us/sample - loss: 0.0421 - accuracy: 0.9178
    Epoch 40/300
    73/73 [==============================] - 0s 152us/sample - loss: 0.0431 - accuracy: 0.9178
    Epoch 41/300
    73/73 [==============================] - 0s 133us/sample - loss: 0.0501 - accuracy: 0.9041
    Epoch 42/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0407 - accuracy: 0.9178
    Epoch 43/300
    73/73 [==============================] - 0s 149us/sample - loss: 0.0389 - accuracy: 0.9178
    Epoch 44/300
    73/73 [==============================] - 0s 143us/sample - loss: 0.0394 - accuracy: 0.9178
    Epoch 45/300
    73/73 [==============================] - 0s 138us/sample - loss: 0.0409 - accuracy: 0.9178
    Epoch 46/300
    73/73 [==============================] - 0s 150us/sample - loss: 0.0403 - accuracy: 0.9178
    Epoch 47/300
    73/73 [==============================] - 0s 149us/sample - loss: 0.0431 - accuracy: 0.9178
    Epoch 48/300
    73/73 [==============================] - 0s 158us/sample - loss: 0.0354 - accuracy: 0.9178
    Epoch 49/300
    73/73 [==============================] - 0s 170us/sample - loss: 0.0420 - accuracy: 0.9178
    Epoch 50/300
    73/73 [==============================] - 0s 142us/sample - loss: 0.0392 - accuracy: 0.9178
    Epoch 51/300
    73/73 [==============================] - 0s 167us/sample - loss: 0.0334 - accuracy: 0.9178
    Epoch 52/300
    73/73 [==============================] - 0s 165us/sample - loss: 0.0352 - accuracy: 0.9178
    Epoch 53/300
    73/73 [==============================] - 0s 129us/sample - loss: 0.0363 - accuracy: 0.9178
    Epoch 54/300
    73/73 [==============================] - 0s 150us/sample - loss: 0.0355 - accuracy: 0.9178
    Epoch 55/300
    73/73 [==============================] - 0s 141us/sample - loss: 0.0373 - accuracy: 0.9178
    Epoch 56/300
    73/73 [==============================] - 0s 129us/sample - loss: 0.0320 - accuracy: 0.9178
    Epoch 57/300
    73/73 [==============================] - 0s 135us/sample - loss: 0.0338 - accuracy: 0.9178
    Epoch 58/300
    73/73 [==============================] - 0s 140us/sample - loss: 0.0332 - accuracy: 0.9178
    Epoch 59/300
    73/73 [==============================] - 0s 132us/sample - loss: 0.0377 - accuracy: 0.9178
    Epoch 60/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0312 - accuracy: 0.9178
    Epoch 61/300
    73/73 [==============================] - 0s 137us/sample - loss: 0.0344 - accuracy: 0.9178
    Epoch 62/300
    73/73 [==============================] - 0s 129us/sample - loss: 0.0332 - accuracy: 0.9178
    Epoch 63/300
    73/73 [==============================] - 0s 135us/sample - loss: 0.0334 - accuracy: 0.9178
    Epoch 64/300
    73/73 [==============================] - 0s 145us/sample - loss: 0.0347 - accuracy: 0.9178
    Epoch 65/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0340 - accuracy: 0.9178
    Epoch 66/300
    73/73 [==============================] - 0s 135us/sample - loss: 0.0370 - accuracy: 0.9178
    Epoch 67/300
    73/73 [==============================] - 0s 144us/sample - loss: 0.0335 - accuracy: 0.9178
    Epoch 68/300
    73/73 [==============================] - 0s 138us/sample - loss: 0.0289 - accuracy: 0.9178
    Epoch 69/300
    73/73 [==============================] - 0s 124us/sample - loss: 0.0328 - accuracy: 0.9178
    Epoch 70/300
    73/73 [==============================] - 0s 141us/sample - loss: 0.0350 - accuracy: 0.9178
    Epoch 71/300
    73/73 [==============================] - 0s 142us/sample - loss: 0.0277 - accuracy: 0.9178
    Epoch 72/300
    73/73 [==============================] - 0s 135us/sample - loss: 0.0272 - accuracy: 0.9178
    Epoch 73/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0292 - accuracy: 0.9178
    Epoch 74/300
    73/73 [==============================] - 0s 146us/sample - loss: 0.0301 - accuracy: 0.9178
    Epoch 75/300
    73/73 [==============================] - 0s 141us/sample - loss: 0.0309 - accuracy: 0.9178
    Epoch 76/300
    73/73 [==============================] - 0s 140us/sample - loss: 0.0269 - accuracy: 0.9178
    Epoch 77/300
    73/73 [==============================] - 0s 143us/sample - loss: 0.0267 - accuracy: 0.9178
    Epoch 78/300
    73/73 [==============================] - 0s 138us/sample - loss: 0.0272 - accuracy: 0.9178
    Epoch 79/300
    73/73 [==============================] - 0s 139us/sample - loss: 0.0318 - accuracy: 0.9178
    Epoch 80/300
    73/73 [==============================] - 0s 138us/sample - loss: 0.0241 - accuracy: 0.9178
    Epoch 81/300
    73/73 [==============================] - 0s 139us/sample - loss: 0.0253 - accuracy: 0.9178
    Epoch 82/300
    73/73 [==============================] - 0s 129us/sample - loss: 0.0248 - accuracy: 0.9178
    Epoch 83/300
    73/73 [==============================] - 0s 138us/sample - loss: 0.0295 - accuracy: 0.9178
    Epoch 84/300
    73/73 [==============================] - 0s 127us/sample - loss: 0.0300 - accuracy: 0.9178
    Epoch 85/300
    73/73 [==============================] - 0s 126us/sample - loss: 0.0220 - accuracy: 0.9315
    Epoch 86/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0266 - accuracy: 0.9178
    Epoch 87/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0279 - accuracy: 0.9178
    Epoch 88/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0318 - accuracy: 0.9178
    Epoch 89/300
    73/73 [==============================] - 0s 133us/sample - loss: 0.0296 - accuracy: 0.9178
    Epoch 90/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0306 - accuracy: 0.9178
    Epoch 91/300
    73/73 [==============================] - 0s 138us/sample - loss: 0.0234 - accuracy: 0.9178
    Epoch 92/300
    73/73 [==============================] - 0s 146us/sample - loss: 0.0294 - accuracy: 0.9315
    Epoch 93/300
    73/73 [==============================] - 0s 125us/sample - loss: 0.0235 - accuracy: 0.9178
    Epoch 94/300
    73/73 [==============================] - 0s 148us/sample - loss: 0.0305 - accuracy: 0.9178
    Epoch 95/300
    73/73 [==============================] - 0s 142us/sample - loss: 0.0320 - accuracy: 0.9041
    Epoch 96/300
    73/73 [==============================] - 0s 124us/sample - loss: 0.0259 - accuracy: 0.9178
    Epoch 97/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0264 - accuracy: 0.9178
    Epoch 98/300
    73/73 [==============================] - 0s 146us/sample - loss: 0.0294 - accuracy: 0.9178
    Epoch 99/300
    73/73 [==============================] - 0s 133us/sample - loss: 0.0248 - accuracy: 0.9178
    Epoch 100/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0256 - accuracy: 0.9178
    Epoch 101/300
    73/73 [==============================] - 0s 152us/sample - loss: 0.0229 - accuracy: 0.9315
    Epoch 102/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0291 - accuracy: 0.9178
    Epoch 103/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0224 - accuracy: 0.9178
    Epoch 104/300
    73/73 [==============================] - 0s 140us/sample - loss: 0.0235 - accuracy: 0.9178
    Epoch 105/300
    73/73 [==============================] - 0s 147us/sample - loss: 0.0277 - accuracy: 0.9041
    Epoch 106/300
    73/73 [==============================] - 0s 125us/sample - loss: 0.0219 - accuracy: 0.9178
    Epoch 107/300
    73/73 [==============================] - 0s 139us/sample - loss: 0.0219 - accuracy: 0.9315
    Epoch 108/300
    73/73 [==============================] - 0s 140us/sample - loss: 0.0253 - accuracy: 0.9178
    Epoch 109/300
    73/73 [==============================] - 0s 127us/sample - loss: 0.0243 - accuracy: 0.9315
    Epoch 110/300
    73/73 [==============================] - 0s 137us/sample - loss: 0.0234 - accuracy: 0.9178
    Epoch 111/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0318 - accuracy: 0.9041
    Epoch 112/300
    73/73 [==============================] - 0s 121us/sample - loss: 0.0215 - accuracy: 0.9178
    Epoch 113/300
    73/73 [==============================] - 0s 129us/sample - loss: 0.0281 - accuracy: 0.9178
    Epoch 114/300
    73/73 [==============================] - 0s 143us/sample - loss: 0.0227 - accuracy: 0.9315
    Epoch 115/300
    73/73 [==============================] - 0s 125us/sample - loss: 0.0270 - accuracy: 0.9178
    Epoch 116/300
    73/73 [==============================] - 0s 133us/sample - loss: 0.0277 - accuracy: 0.9178
    Epoch 117/300
    73/73 [==============================] - 0s 139us/sample - loss: 0.0308 - accuracy: 0.9178
    Epoch 118/300
    73/73 [==============================] - 0s 129us/sample - loss: 0.0287 - accuracy: 0.9315
    Epoch 119/300
    73/73 [==============================] - 0s 141us/sample - loss: 0.0218 - accuracy: 0.9178
    Epoch 120/300
    73/73 [==============================] - 0s 143us/sample - loss: 0.0239 - accuracy: 0.9178
    Epoch 121/300
    73/73 [==============================] - 0s 167us/sample - loss: 0.0254 - accuracy: 0.9178
    Epoch 122/300
    73/73 [==============================] - 0s 172us/sample - loss: 0.0218 - accuracy: 0.9178
    Epoch 123/300
    73/73 [==============================] - 0s 174us/sample - loss: 0.0221 - accuracy: 0.9178
    Epoch 124/300
    73/73 [==============================] - 0s 163us/sample - loss: 0.0272 - accuracy: 0.9178
    Epoch 125/300
    73/73 [==============================] - 0s 146us/sample - loss: 0.0216 - accuracy: 0.9178
    Epoch 126/300
    73/73 [==============================] - 0s 148us/sample - loss: 0.0231 - accuracy: 0.9178
    Epoch 127/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0228 - accuracy: 0.9178
    Epoch 128/300
    73/73 [==============================] - 0s 144us/sample - loss: 0.0219 - accuracy: 0.9178
    Epoch 129/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0241 - accuracy: 0.9178
    Epoch 130/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0245 - accuracy: 0.9178
    Epoch 131/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0233 - accuracy: 0.9315
    Epoch 132/300
    73/73 [==============================] - 0s 135us/sample - loss: 0.0211 - accuracy: 0.9178
    Epoch 133/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0221 - accuracy: 0.9178
    Epoch 134/300
    73/73 [==============================] - 0s 135us/sample - loss: 0.0244 - accuracy: 0.9178
    Epoch 135/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0223 - accuracy: 0.9315
    Epoch 136/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0332 - accuracy: 0.9041
    Epoch 137/300
    73/73 [==============================] - 0s 139us/sample - loss: 0.0217 - accuracy: 0.9178
    Epoch 138/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0253 - accuracy: 0.9178
    Epoch 139/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0203 - accuracy: 0.9178
    Epoch 140/300
    73/73 [==============================] - 0s 145us/sample - loss: 0.0219 - accuracy: 0.9178
    Epoch 141/300
    73/73 [==============================] - 0s 139us/sample - loss: 0.0281 - accuracy: 0.9178
    Epoch 142/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0206 - accuracy: 0.9178
    Epoch 143/300
    73/73 [==============================] - 0s 143us/sample - loss: 0.0269 - accuracy: 0.9041
    Epoch 144/300
    73/73 [==============================] - 0s 144us/sample - loss: 0.0293 - accuracy: 0.9178
    Epoch 145/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0290 - accuracy: 0.9178
    Epoch 146/300
    73/73 [==============================] - 0s 125us/sample - loss: 0.0198 - accuracy: 0.9178
    Epoch 147/300
    73/73 [==============================] - 0s 137us/sample - loss: 0.0242 - accuracy: 0.9178
    Epoch 148/300
    73/73 [==============================] - 0s 135us/sample - loss: 0.0218 - accuracy: 0.9178
    Epoch 149/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0267 - accuracy: 0.9041
    Epoch 150/300
    73/73 [==============================] - 0s 149us/sample - loss: 0.0221 - accuracy: 0.9178
    Epoch 151/300
    73/73 [==============================] - 0s 139us/sample - loss: 0.0222 - accuracy: 0.9178
    Epoch 152/300
    73/73 [==============================] - 0s 132us/sample - loss: 0.0225 - accuracy: 0.9315
    Epoch 153/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0240 - accuracy: 0.9315
    Epoch 154/300
    73/73 [==============================] - 0s 129us/sample - loss: 0.0218 - accuracy: 0.9178
    Epoch 155/300
    73/73 [==============================] - 0s 133us/sample - loss: 0.0282 - accuracy: 0.9178
    Epoch 156/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0226 - accuracy: 0.9178
    Epoch 157/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0232 - accuracy: 0.9178
    Epoch 158/300
    73/73 [==============================] - 0s 133us/sample - loss: 0.0217 - accuracy: 0.9178
    Epoch 159/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0191 - accuracy: 0.9178
    Epoch 160/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0214 - accuracy: 0.9178
    Epoch 161/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0201 - accuracy: 0.9178
    Epoch 162/300
    73/73 [==============================] - 0s 126us/sample - loss: 0.0233 - accuracy: 0.9178
    Epoch 163/300
    73/73 [==============================] - 0s 133us/sample - loss: 0.0217 - accuracy: 0.9178
    Epoch 164/300
    73/73 [==============================] - 0s 140us/sample - loss: 0.0189 - accuracy: 0.9178
    Epoch 165/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0192 - accuracy: 0.9178
    Epoch 166/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0230 - accuracy: 0.9178
    Epoch 167/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0235 - accuracy: 0.9178
    Epoch 168/300
    73/73 [==============================] - 0s 142us/sample - loss: 0.0185 - accuracy: 0.9178
    Epoch 169/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0304 - accuracy: 0.9041
    Epoch 170/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0198 - accuracy: 0.9178
    Epoch 171/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0215 - accuracy: 0.9178
    Epoch 172/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0243 - accuracy: 0.9178
    Epoch 173/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0256 - accuracy: 0.9178
    Epoch 174/300
    73/73 [==============================] - 0s 132us/sample - loss: 0.0239 - accuracy: 0.9178
    Epoch 175/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0205 - accuracy: 0.9178
    Epoch 176/300
    73/73 [==============================] - 0s 138us/sample - loss: 0.0185 - accuracy: 0.9178
    Epoch 177/300
    73/73 [==============================] - 0s 151us/sample - loss: 0.0261 - accuracy: 0.9178
    Epoch 178/300
    73/73 [==============================] - 0s 135us/sample - loss: 0.0203 - accuracy: 0.9315
    Epoch 179/300
    73/73 [==============================] - 0s 152us/sample - loss: 0.0225 - accuracy: 0.9178
    Epoch 180/300
    73/73 [==============================] - 0s 126us/sample - loss: 0.0236 - accuracy: 0.9178
    Epoch 181/300
    73/73 [==============================] - 0s 137us/sample - loss: 0.0207 - accuracy: 0.9178
    Epoch 182/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0218 - accuracy: 0.9178
    Epoch 183/300
    73/73 [==============================] - 0s 135us/sample - loss: 0.0193 - accuracy: 0.9178
    Epoch 184/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0184 - accuracy: 0.9315
    Epoch 185/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0186 - accuracy: 0.9178
    Epoch 186/300
    73/73 [==============================] - 0s 135us/sample - loss: 0.0233 - accuracy: 0.9178
    Epoch 187/300
    73/73 [==============================] - 0s 141us/sample - loss: 0.0192 - accuracy: 0.9178
    Epoch 188/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0252 - accuracy: 0.9041
    Epoch 189/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0246 - accuracy: 0.9178
    Epoch 190/300
    73/73 [==============================] - 0s 145us/sample - loss: 0.0221 - accuracy: 0.9315
    Epoch 191/300
    73/73 [==============================] - 0s 143us/sample - loss: 0.0218 - accuracy: 0.9178
    Epoch 192/300
    73/73 [==============================] - 0s 153us/sample - loss: 0.0205 - accuracy: 0.9178
    Epoch 193/300
    73/73 [==============================] - 0s 142us/sample - loss: 0.0255 - accuracy: 0.9178
    Epoch 194/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0202 - accuracy: 0.9178
    Epoch 195/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0178 - accuracy: 0.9315
    Epoch 196/300
    73/73 [==============================] - 0s 145us/sample - loss: 0.0193 - accuracy: 0.9315
    Epoch 197/300
    73/73 [==============================] - 0s 127us/sample - loss: 0.0206 - accuracy: 0.9315
    Epoch 198/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0202 - accuracy: 0.9178
    Epoch 199/300
    73/73 [==============================] - 0s 129us/sample - loss: 0.0283 - accuracy: 0.9178
    Epoch 200/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0263 - accuracy: 0.9178
    Epoch 201/300
    73/73 [==============================] - 0s 129us/sample - loss: 0.0202 - accuracy: 0.9178
    Epoch 202/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0241 - accuracy: 0.9178
    Epoch 203/300
    73/73 [==============================] - 0s 123us/sample - loss: 0.0231 - accuracy: 0.9315
    Epoch 204/300
    73/73 [==============================] - 0s 126us/sample - loss: 0.0214 - accuracy: 0.9178
    Epoch 205/300
    73/73 [==============================] - 0s 132us/sample - loss: 0.0252 - accuracy: 0.9178
    Epoch 206/300
    73/73 [==============================] - 0s 126us/sample - loss: 0.0215 - accuracy: 0.9178
    Epoch 207/300
    73/73 [==============================] - 0s 127us/sample - loss: 0.0258 - accuracy: 0.9178
    Epoch 208/300
    73/73 [==============================] - 0s 132us/sample - loss: 0.0239 - accuracy: 0.9178
    Epoch 209/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0240 - accuracy: 0.9178
    Epoch 210/300
    73/73 [==============================] - 0s 133us/sample - loss: 0.0218 - accuracy: 0.9178
    Epoch 211/300
    73/73 [==============================] - ETA: 0s - loss: 0.0356 - accuracy: 0.86 - 0s 138us/sample - loss: 0.0184 - accuracy: 0.9315
    Epoch 212/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0298 - accuracy: 0.9178
    Epoch 213/300
    73/73 [==============================] - 0s 127us/sample - loss: 0.0211 - accuracy: 0.9178
    Epoch 214/300
    73/73 [==============================] - 0s 145us/sample - loss: 0.0238 - accuracy: 0.9178
    Epoch 215/300
    73/73 [==============================] - 0s 139us/sample - loss: 0.0247 - accuracy: 0.9315
    Epoch 216/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0232 - accuracy: 0.9178
    Epoch 217/300
    73/73 [==============================] - 0s 148us/sample - loss: 0.0230 - accuracy: 0.9178
    Epoch 218/300
    73/73 [==============================] - 0s 143us/sample - loss: 0.0227 - accuracy: 0.9178
    Epoch 219/300
    73/73 [==============================] - 0s 137us/sample - loss: 0.0234 - accuracy: 0.9178
    Epoch 220/300
    73/73 [==============================] - 0s 133us/sample - loss: 0.0213 - accuracy: 0.9178
    Epoch 221/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0199 - accuracy: 0.9178
    Epoch 222/300
    73/73 [==============================] - 0s 124us/sample - loss: 0.0208 - accuracy: 0.9178
    Epoch 223/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0222 - accuracy: 0.9178
    Epoch 224/300
    73/73 [==============================] - 0s 138us/sample - loss: 0.0293 - accuracy: 0.9178
    Epoch 225/300
    73/73 [==============================] - 0s 123us/sample - loss: 0.0230 - accuracy: 0.9178
    Epoch 226/300
    73/73 [==============================] - 0s 137us/sample - loss: 0.0227 - accuracy: 0.9178
    Epoch 227/300
    73/73 [==============================] - 0s 135us/sample - loss: 0.0258 - accuracy: 0.9315
    Epoch 228/300
    73/73 [==============================] - 0s 143us/sample - loss: 0.0209 - accuracy: 0.9178
    Epoch 229/300
    73/73 [==============================] - 0s 132us/sample - loss: 0.0219 - accuracy: 0.9178
    Epoch 230/300
    73/73 [==============================] - 0s 141us/sample - loss: 0.0223 - accuracy: 0.9178
    Epoch 231/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0259 - accuracy: 0.9178
    Epoch 232/300
    73/73 [==============================] - 0s 129us/sample - loss: 0.0231 - accuracy: 0.9178
    Epoch 233/300
    73/73 [==============================] - 0s 145us/sample - loss: 0.0199 - accuracy: 0.9178
    Epoch 234/300
    73/73 [==============================] - 0s 138us/sample - loss: 0.0260 - accuracy: 0.9178
    Epoch 235/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0195 - accuracy: 0.9178
    Epoch 236/300
    73/73 [==============================] - 0s 132us/sample - loss: 0.0214 - accuracy: 0.9178
    Epoch 237/300
    73/73 [==============================] - 0s 135us/sample - loss: 0.0244 - accuracy: 0.9178
    Epoch 238/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0228 - accuracy: 0.9178
    Epoch 239/300
    73/73 [==============================] - 0s 132us/sample - loss: 0.0214 - accuracy: 0.9178
    Epoch 240/300
    73/73 [==============================] - 0s 129us/sample - loss: 0.0260 - accuracy: 0.9041
    Epoch 241/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0224 - accuracy: 0.9315
    Epoch 242/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0179 - accuracy: 0.9178
    Epoch 243/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0210 - accuracy: 0.9178
    Epoch 244/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0194 - accuracy: 0.9178
    Epoch 245/300
    73/73 [==============================] - ETA: 0s - loss: 9.4358e-04 - accuracy: 1.00 - 0s 134us/sample - loss: 0.0238 - accuracy: 0.9178
    Epoch 246/300
    73/73 [==============================] - ETA: 0s - loss: 0.0306 - accuracy: 0.93 - 0s 132us/sample - loss: 0.0246 - accuracy: 0.9178
    Epoch 247/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0236 - accuracy: 0.9178
    Epoch 248/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0217 - accuracy: 0.9178
    Epoch 249/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0269 - accuracy: 0.9178
    Epoch 250/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0212 - accuracy: 0.9178
    Epoch 251/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0231 - accuracy: 0.9178
    Epoch 252/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0204 - accuracy: 0.9178
    Epoch 253/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0168 - accuracy: 0.9178
    Epoch 254/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0236 - accuracy: 0.9178
    Epoch 255/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0206 - accuracy: 0.9178
    Epoch 256/300
    73/73 [==============================] - 0s 126us/sample - loss: 0.0222 - accuracy: 0.9178
    Epoch 257/300
    73/73 [==============================] - 0s 143us/sample - loss: 0.0223 - accuracy: 0.9178
    Epoch 258/300
    73/73 [==============================] - 0s 132us/sample - loss: 0.0247 - accuracy: 0.9178
    Epoch 259/300
    73/73 [==============================] - 0s 127us/sample - loss: 0.0229 - accuracy: 0.9178
    Epoch 260/300
    73/73 [==============================] - 0s 133us/sample - loss: 0.0201 - accuracy: 0.9178
    Epoch 261/300
    73/73 [==============================] - 0s 135us/sample - loss: 0.0197 - accuracy: 0.9178
    Epoch 262/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0230 - accuracy: 0.9178
    Epoch 263/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0188 - accuracy: 0.9178
    Epoch 264/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0213 - accuracy: 0.9178
    Epoch 265/300
    73/73 [==============================] - 0s 124us/sample - loss: 0.0196 - accuracy: 0.9178
    Epoch 266/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0225 - accuracy: 0.9178
    Epoch 267/300
    73/73 [==============================] - 0s 145us/sample - loss: 0.0227 - accuracy: 0.9178
    Epoch 268/300
    73/73 [==============================] - 0s 133us/sample - loss: 0.0168 - accuracy: 0.9178
    Epoch 269/300
    73/73 [==============================] - 0s 135us/sample - loss: 0.0214 - accuracy: 0.9178
    Epoch 270/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0223 - accuracy: 0.9178
    Epoch 271/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0207 - accuracy: 0.9178
    Epoch 272/300
    73/73 [==============================] - 0s 132us/sample - loss: 0.0225 - accuracy: 0.9178
    Epoch 273/300
    73/73 [==============================] - 0s 133us/sample - loss: 0.0200 - accuracy: 0.9178
    Epoch 274/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0225 - accuracy: 0.9178
    Epoch 275/300
    73/73 [==============================] - 0s 150us/sample - loss: 0.0271 - accuracy: 0.9178
    Epoch 276/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0204 - accuracy: 0.9178
    Epoch 277/300
    73/73 [==============================] - 0s 138us/sample - loss: 0.0249 - accuracy: 0.9178
    Epoch 278/300
    73/73 [==============================] - 0s 134us/sample - loss: 0.0227 - accuracy: 0.9178
    Epoch 279/300
    73/73 [==============================] - 0s 139us/sample - loss: 0.0240 - accuracy: 0.9178
    Epoch 280/300
    73/73 [==============================] - 0s 133us/sample - loss: 0.0219 - accuracy: 0.9178
    Epoch 281/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0257 - accuracy: 0.9041
    Epoch 282/300
    73/73 [==============================] - 0s 141us/sample - loss: 0.0187 - accuracy: 0.9178
    Epoch 283/300
    73/73 [==============================] - 0s 129us/sample - loss: 0.0199 - accuracy: 0.9178
    Epoch 284/300
    73/73 [==============================] - 0s 131us/sample - loss: 0.0192 - accuracy: 0.9178
    Epoch 285/300
    73/73 [==============================] - 0s 139us/sample - loss: 0.0205 - accuracy: 0.9178
    Epoch 286/300
    73/73 [==============================] - 0s 130us/sample - loss: 0.0214 - accuracy: 0.9178
    Epoch 287/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0219 - accuracy: 0.9178
    Epoch 288/300
    73/73 [==============================] - 0s 132us/sample - loss: 0.0220 - accuracy: 0.9178
    Epoch 289/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0210 - accuracy: 0.9178
    Epoch 290/300
    73/73 [==============================] - 0s 195us/sample - loss: 0.0199 - accuracy: 0.9178
    Epoch 291/300
    73/73 [==============================] - 0s 154us/sample - loss: 0.0227 - accuracy: 0.9178
    Epoch 292/300
    73/73 [==============================] - ETA: 0s - loss: 0.0282 - accuracy: 0.80 - 0s 150us/sample - loss: 0.0180 - accuracy: 0.9178
    Epoch 293/300
    73/73 [==============================] - 0s 184us/sample - loss: 0.0177 - accuracy: 0.9178
    Epoch 294/300
    73/73 [==============================] - 0s 144us/sample - loss: 0.0222 - accuracy: 0.9315
    Epoch 295/300
    73/73 [==============================] - 0s 133us/sample - loss: 0.0180 - accuracy: 0.9178
    Epoch 296/300
    73/73 [==============================] - 0s 133us/sample - loss: 0.0214 - accuracy: 0.9178
    Epoch 297/300
    73/73 [==============================] - 0s 141us/sample - loss: 0.0206 - accuracy: 0.9178
    Epoch 298/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0208 - accuracy: 0.9178
    Epoch 299/300
    73/73 [==============================] - 0s 128us/sample - loss: 0.0222 - accuracy: 0.9178
    Epoch 300/300
    73/73 [==============================] - 0s 136us/sample - loss: 0.0190 - accuracy: 0.9178
    Model: "sequential_13"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    dense_32 (Dense)             (None, 128)               4608      
    _________________________________________________________________
    activation_32 (Activation)   (None, 128)               0         
    _________________________________________________________________
    dropout_21 (Dropout)         (None, 128)               0         
    _________________________________________________________________
    dense_33 (Dense)             (None, 1)                 129       
    _________________________________________________________________
    activation_33 (Activation)   (None, 1)                 0         
    =================================================================
    Total params: 4,737
    Trainable params: 4,737
    Non-trainable params: 0
    _________________________________________________________________
    Train on 74 samples
    Epoch 1/300
    74/74 [==============================] - 1s 7ms/sample - loss: 0.1509 - accuracy: 0.3784
    Epoch 2/300
    74/74 [==============================] - 0s 144us/sample - loss: 0.1408 - accuracy: 0.4189
    Epoch 3/300
    74/74 [==============================] - 0s 135us/sample - loss: 0.1246 - accuracy: 0.5811
    Epoch 4/300
    74/74 [==============================] - 0s 138us/sample - loss: 0.1236 - accuracy: 0.6351
    Epoch 5/300
    74/74 [==============================] - 0s 130us/sample - loss: 0.1165 - accuracy: 0.6622
    Epoch 6/300
    74/74 [==============================] - 0s 136us/sample - loss: 0.1111 - accuracy: 0.7027
    Epoch 7/300
    74/74 [==============================] - 0s 138us/sample - loss: 0.1085 - accuracy: 0.7297
    Epoch 8/300
    74/74 [==============================] - 0s 130us/sample - loss: 0.1057 - accuracy: 0.7973
    Epoch 9/300
    74/74 [==============================] - 0s 129us/sample - loss: 0.1005 - accuracy: 0.8108
    Epoch 10/300
    74/74 [==============================] - 0s 132us/sample - loss: 0.1018 - accuracy: 0.8243
    Epoch 11/300
    74/74 [==============================] - 0s 132us/sample - loss: 0.0886 - accuracy: 0.8108
    Epoch 12/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0900 - accuracy: 0.8649
    Epoch 13/300
    74/74 [==============================] - 0s 129us/sample - loss: 0.0827 - accuracy: 0.8784
    Epoch 14/300
    74/74 [==============================] - 0s 131us/sample - loss: 0.0843 - accuracy: 0.8514
    Epoch 15/300
    74/74 [==============================] - 0s 132us/sample - loss: 0.0771 - accuracy: 0.8784
    Epoch 16/300
    74/74 [==============================] - 0s 132us/sample - loss: 0.0774 - accuracy: 0.8919
    Epoch 17/300
    74/74 [==============================] - 0s 131us/sample - loss: 0.0703 - accuracy: 0.8784
    Epoch 18/300
    74/74 [==============================] - 0s 130us/sample - loss: 0.0686 - accuracy: 0.8784
    Epoch 19/300
    74/74 [==============================] - 0s 144us/sample - loss: 0.0724 - accuracy: 0.8784
    Epoch 20/300
    74/74 [==============================] - 0s 137us/sample - loss: 0.0600 - accuracy: 0.8919
    Epoch 21/300
    74/74 [==============================] - 0s 136us/sample - loss: 0.0621 - accuracy: 0.8784
    Epoch 22/300
    74/74 [==============================] - 0s 142us/sample - loss: 0.0650 - accuracy: 0.8784
    Epoch 23/300
    74/74 [==============================] - 0s 137us/sample - loss: 0.0611 - accuracy: 0.8784
    Epoch 24/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0568 - accuracy: 0.8784
    Epoch 25/300
    74/74 [==============================] - 0s 144us/sample - loss: 0.0544 - accuracy: 0.8784
    Epoch 26/300
    74/74 [==============================] - 0s 135us/sample - loss: 0.0543 - accuracy: 0.8919
    Epoch 27/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0568 - accuracy: 0.8784
    Epoch 28/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0595 - accuracy: 0.8784
    Epoch 29/300
    74/74 [==============================] - 0s 135us/sample - loss: 0.0570 - accuracy: 0.8784
    Epoch 30/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0481 - accuracy: 0.8784
    Epoch 31/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0461 - accuracy: 0.8784
    Epoch 32/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0500 - accuracy: 0.8784
    Epoch 33/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0429 - accuracy: 0.8919
    Epoch 34/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0440 - accuracy: 0.8784
    Epoch 35/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0505 - accuracy: 0.8784
    Epoch 36/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0483 - accuracy: 0.8784
    Epoch 37/300
    74/74 [==============================] - 0s 131us/sample - loss: 0.0430 - accuracy: 0.8784
    Epoch 38/300
    74/74 [==============================] - 0s 132us/sample - loss: 0.0395 - accuracy: 0.8919
    Epoch 39/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0417 - accuracy: 0.8784
    Epoch 40/300
    74/74 [==============================] - 0s 133us/sample - loss: 0.0424 - accuracy: 0.8919
    Epoch 41/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0397 - accuracy: 0.8919
    Epoch 42/300
    74/74 [==============================] - 0s 139us/sample - loss: 0.0396 - accuracy: 0.8919
    Epoch 43/300
    74/74 [==============================] - 0s 133us/sample - loss: 0.0325 - accuracy: 0.8784
    Epoch 44/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0402 - accuracy: 0.8919
    Epoch 45/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0396 - accuracy: 0.8919
    Epoch 46/300
    74/74 [==============================] - 0s 133us/sample - loss: 0.0361 - accuracy: 0.8919
    Epoch 47/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0377 - accuracy: 0.8919
    Epoch 48/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0379 - accuracy: 0.8919
    Epoch 49/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0373 - accuracy: 0.8919
    Epoch 50/300
    74/74 [==============================] - 0s 117us/sample - loss: 0.0379 - accuracy: 0.8919
    Epoch 51/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0380 - accuracy: 0.8919
    Epoch 52/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0398 - accuracy: 0.8919
    Epoch 53/300
    74/74 [==============================] - 0s 129us/sample - loss: 0.0424 - accuracy: 0.8784
    Epoch 54/300
    74/74 [==============================] - 0s 129us/sample - loss: 0.0390 - accuracy: 0.8919
    Epoch 55/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0322 - accuracy: 0.8919
    Epoch 56/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0310 - accuracy: 0.8919
    Epoch 57/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0354 - accuracy: 0.8919
    Epoch 58/300
    74/74 [==============================] - 0s 142us/sample - loss: 0.0365 - accuracy: 0.8919
    Epoch 59/300
    74/74 [==============================] - 0s 132us/sample - loss: 0.0312 - accuracy: 0.8919
    Epoch 60/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0396 - accuracy: 0.8919
    Epoch 61/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0326 - accuracy: 0.8919
    Epoch 62/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0389 - accuracy: 0.8919
    Epoch 63/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0311 - accuracy: 0.8919
    Epoch 64/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0341 - accuracy: 0.8919
    Epoch 65/300
    74/74 [==============================] - ETA: 0s - loss: 0.0291 - accuracy: 0.86 - 0s 138us/sample - loss: 0.0308 - accuracy: 0.8919
    Epoch 66/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0383 - accuracy: 0.8919
    Epoch 67/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0340 - accuracy: 0.8919
    Epoch 68/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0321 - accuracy: 0.8919
    Epoch 69/300
    74/74 [==============================] - 0s 120us/sample - loss: 0.0310 - accuracy: 0.8919
    Epoch 70/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0316 - accuracy: 0.8919
    Epoch 71/300
    74/74 [==============================] - 0s 120us/sample - loss: 0.0296 - accuracy: 0.8919
    Epoch 72/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0406 - accuracy: 0.8919
    Epoch 73/300
    74/74 [==============================] - 0s 129us/sample - loss: 0.0283 - accuracy: 0.8919
    Epoch 74/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0326 - accuracy: 0.8919
    Epoch 75/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0310 - accuracy: 0.8919
    Epoch 76/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0299 - accuracy: 0.8919
    Epoch 77/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0341 - accuracy: 0.8919
    Epoch 78/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0316 - accuracy: 0.8919
    Epoch 79/300
    74/74 [==============================] - 0s 131us/sample - loss: 0.0306 - accuracy: 0.8919
    Epoch 80/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0365 - accuracy: 0.8919
    Epoch 81/300
    74/74 [==============================] - 0s 129us/sample - loss: 0.0367 - accuracy: 0.8919
    Epoch 82/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0322 - accuracy: 0.8919
    Epoch 83/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0370 - accuracy: 0.8919
    Epoch 84/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0311 - accuracy: 0.8919
    Epoch 85/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0341 - accuracy: 0.8919
    Epoch 86/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0275 - accuracy: 0.9054
    Epoch 87/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0279 - accuracy: 0.8919
    Epoch 88/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0300 - accuracy: 0.8919
    Epoch 89/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0319 - accuracy: 0.8919
    Epoch 90/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0285 - accuracy: 0.8919
    Epoch 91/300
    74/74 [==============================] - 0s 119us/sample - loss: 0.0328 - accuracy: 0.8919
    Epoch 92/300
    74/74 [==============================] - 0s 131us/sample - loss: 0.0316 - accuracy: 0.8919
    Epoch 93/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0355 - accuracy: 0.8919
    Epoch 94/300
    74/74 [==============================] - 0s 120us/sample - loss: 0.0332 - accuracy: 0.8919
    Epoch 95/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0370 - accuracy: 0.8919
    Epoch 96/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0320 - accuracy: 0.8919
    Epoch 97/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0365 - accuracy: 0.8919
    Epoch 98/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0280 - accuracy: 0.8919
    Epoch 99/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0303 - accuracy: 0.8919
    Epoch 100/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0293 - accuracy: 0.8919
    Epoch 101/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0316 - accuracy: 0.8919
    Epoch 102/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0334 - accuracy: 0.8919
    Epoch 103/300
    74/74 [==============================] - 0s 134us/sample - loss: 0.0285 - accuracy: 0.8919
    Epoch 104/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0320 - accuracy: 0.8919
    Epoch 105/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0290 - accuracy: 0.8919
    Epoch 106/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0311 - accuracy: 0.8919
    Epoch 107/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0303 - accuracy: 0.8919
    Epoch 108/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0286 - accuracy: 0.8919
    Epoch 109/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0297 - accuracy: 0.8919
    Epoch 110/300
    74/74 [==============================] - ETA: 0s - loss: 0.0160 - accuracy: 1.00 - 0s 121us/sample - loss: 0.0320 - accuracy: 0.8919
    Epoch 111/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0294 - accuracy: 0.8919
    Epoch 112/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0294 - accuracy: 0.8919
    Epoch 113/300
    74/74 [==============================] - 0s 134us/sample - loss: 0.0323 - accuracy: 0.8919
    Epoch 114/300
    74/74 [==============================] - 0s 129us/sample - loss: 0.0307 - accuracy: 0.8919
    Epoch 115/300
    74/74 [==============================] - 0s 141us/sample - loss: 0.0320 - accuracy: 0.8919
    Epoch 116/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0285 - accuracy: 0.8919
    Epoch 117/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0288 - accuracy: 0.8919
    Epoch 118/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0332 - accuracy: 0.8919
    Epoch 119/300
    74/74 [==============================] - 0s 118us/sample - loss: 0.0336 - accuracy: 0.8919
    Epoch 120/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0276 - accuracy: 0.8919
    Epoch 121/300
    74/74 [==============================] - 0s 119us/sample - loss: 0.0306 - accuracy: 0.8919
    Epoch 122/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0334 - accuracy: 0.8919
    Epoch 123/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0283 - accuracy: 0.8919
    Epoch 124/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0328 - accuracy: 0.8919
    Epoch 125/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0317 - accuracy: 0.8919
    Epoch 126/300
    74/74 [==============================] - 0s 132us/sample - loss: 0.0277 - accuracy: 0.8919
    Epoch 127/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0292 - accuracy: 0.8919
    Epoch 128/300
    74/74 [==============================] - 0s 135us/sample - loss: 0.0286 - accuracy: 0.8919
    Epoch 129/300
    74/74 [==============================] - 0s 132us/sample - loss: 0.0300 - accuracy: 0.8919
    Epoch 130/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0374 - accuracy: 0.8919
    Epoch 131/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0295 - accuracy: 0.8919
    Epoch 132/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0294 - accuracy: 0.8919
    Epoch 133/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0303 - accuracy: 0.8919
    Epoch 134/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0311 - accuracy: 0.8919
    Epoch 135/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0352 - accuracy: 0.8784
    Epoch 136/300
    74/74 [==============================] - 0s 130us/sample - loss: 0.0294 - accuracy: 0.8919
    Epoch 137/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0288 - accuracy: 0.8919
    Epoch 138/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0333 - accuracy: 0.8919
    Epoch 139/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0289 - accuracy: 0.8919
    Epoch 140/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0303 - accuracy: 0.8919
    Epoch 141/300
    74/74 [==============================] - 0s 119us/sample - loss: 0.0319 - accuracy: 0.8919
    Epoch 142/300
    74/74 [==============================] - 0s 141us/sample - loss: 0.0311 - accuracy: 0.8919
    Epoch 143/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0295 - accuracy: 0.8919
    Epoch 144/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0299 - accuracy: 0.8919
    Epoch 145/300
    74/74 [==============================] - 0s 140us/sample - loss: 0.0269 - accuracy: 0.8919
    Epoch 146/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0319 - accuracy: 0.8919
    Epoch 147/300
    74/74 [==============================] - 0s 119us/sample - loss: 0.0294 - accuracy: 0.8919
    Epoch 148/300
    74/74 [==============================] - 0s 130us/sample - loss: 0.0284 - accuracy: 0.8919
    Epoch 149/300
    74/74 [==============================] - 0s 120us/sample - loss: 0.0281 - accuracy: 0.8919
    Epoch 150/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0313 - accuracy: 0.8919
    Epoch 151/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0292 - accuracy: 0.8919
    Epoch 152/300
    74/74 [==============================] - ETA: 0s - loss: 0.0309 - accuracy: 0.80 - 0s 115us/sample - loss: 0.0264 - accuracy: 0.8919
    Epoch 153/300
    74/74 [==============================] - 0s 134us/sample - loss: 0.0300 - accuracy: 0.8919
    Epoch 154/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0276 - accuracy: 0.8919
    Epoch 155/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0270 - accuracy: 0.8919
    Epoch 156/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0329 - accuracy: 0.8919
    Epoch 157/300
    74/74 [==============================] - 0s 129us/sample - loss: 0.0337 - accuracy: 0.8919
    Epoch 158/300
    74/74 [==============================] - 0s 118us/sample - loss: 0.0289 - accuracy: 0.8919
    Epoch 159/300
    74/74 [==============================] - 0s 119us/sample - loss: 0.0318 - accuracy: 0.8919
    Epoch 160/300
    74/74 [==============================] - 0s 120us/sample - loss: 0.0248 - accuracy: 0.8919
    Epoch 161/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0293 - accuracy: 0.8919
    Epoch 162/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0261 - accuracy: 0.8919
    Epoch 163/300
    74/74 [==============================] - 0s 119us/sample - loss: 0.0298 - accuracy: 0.8919
    Epoch 164/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0330 - accuracy: 0.8919
    Epoch 165/300
    74/74 [==============================] - 0s 130us/sample - loss: 0.0296 - accuracy: 0.8919
    Epoch 166/300
    74/74 [==============================] - 0s 130us/sample - loss: 0.0282 - accuracy: 0.8919
    Epoch 167/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0272 - accuracy: 0.8919
    Epoch 168/300
    74/74 [==============================] - 0s 132us/sample - loss: 0.0337 - accuracy: 0.8919
    Epoch 169/300
    74/74 [==============================] - 0s 118us/sample - loss: 0.0350 - accuracy: 0.8919
    Epoch 170/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0339 - accuracy: 0.8919
    Epoch 171/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0260 - accuracy: 0.8919
    Epoch 172/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0299 - accuracy: 0.8919
    Epoch 173/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0341 - accuracy: 0.8919
    Epoch 174/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0308 - accuracy: 0.8919
    Epoch 175/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0268 - accuracy: 0.8919
    Epoch 176/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0303 - accuracy: 0.8919
    Epoch 177/300
    74/74 [==============================] - 0s 118us/sample - loss: 0.0279 - accuracy: 0.8919
    Epoch 178/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0308 - accuracy: 0.8919
    Epoch 179/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0298 - accuracy: 0.8919
    Epoch 180/300
    74/74 [==============================] - 0s 119us/sample - loss: 0.0282 - accuracy: 0.8919
    Epoch 181/300
    74/74 [==============================] - 0s 129us/sample - loss: 0.0322 - accuracy: 0.8919
    Epoch 182/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0268 - accuracy: 0.8919
    Epoch 183/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0267 - accuracy: 0.8919
    Epoch 184/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0300 - accuracy: 0.8919
    Epoch 185/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0289 - accuracy: 0.8919
    Epoch 186/300
    74/74 [==============================] - 0s 130us/sample - loss: 0.0296 - accuracy: 0.8919
    Epoch 187/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0283 - accuracy: 0.8919
    Epoch 188/300
    74/74 [==============================] - 0s 120us/sample - loss: 0.0298 - accuracy: 0.8919
    Epoch 189/300
    74/74 [==============================] - ETA: 0s - loss: 0.0218 - accuracy: 0.86 - 0s 126us/sample - loss: 0.0260 - accuracy: 0.8919
    Epoch 190/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0274 - accuracy: 0.8919
    Epoch 191/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0280 - accuracy: 0.8919
    Epoch 192/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0258 - accuracy: 0.8919
    Epoch 193/300
    74/74 [==============================] - ETA: 0s - loss: 0.0083 - accuracy: 0.93 - 0s 121us/sample - loss: 0.0284 - accuracy: 0.8919
    Epoch 194/300
    74/74 [==============================] - 0s 130us/sample - loss: 0.0300 - accuracy: 0.8919
    Epoch 195/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0269 - accuracy: 0.8919
    Epoch 196/300
    74/74 [==============================] - 0s 120us/sample - loss: 0.0269 - accuracy: 0.8919
    Epoch 197/300
    74/74 [==============================] - 0s 137us/sample - loss: 0.0308 - accuracy: 0.8919
    Epoch 198/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0243 - accuracy: 0.8919
    Epoch 199/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0280 - accuracy: 0.8919
    Epoch 200/300
    74/74 [==============================] - 0s 130us/sample - loss: 0.0269 - accuracy: 0.9054
    Epoch 201/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0312 - accuracy: 0.8919
    Epoch 202/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0324 - accuracy: 0.8919
    Epoch 203/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0298 - accuracy: 0.8919
    Epoch 204/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0314 - accuracy: 0.8919
    Epoch 205/300
    74/74 [==============================] - 0s 119us/sample - loss: 0.0286 - accuracy: 0.8919
    Epoch 206/300
    74/74 [==============================] - 0s 132us/sample - loss: 0.0298 - accuracy: 0.8919
    Epoch 207/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0320 - accuracy: 0.8919
    Epoch 208/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0360 - accuracy: 0.8919
    Epoch 209/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0308 - accuracy: 0.8919
    Epoch 210/300
    74/74 [==============================] - 0s 120us/sample - loss: 0.0280 - accuracy: 0.8919
    Epoch 211/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0342 - accuracy: 0.8919
    Epoch 212/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0311 - accuracy: 0.8919
    Epoch 213/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0269 - accuracy: 0.8919
    Epoch 214/300
    74/74 [==============================] - 0s 129us/sample - loss: 0.0285 - accuracy: 0.8919
    Epoch 215/300
    74/74 [==============================] - 0s 129us/sample - loss: 0.0290 - accuracy: 0.8919
    Epoch 216/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0260 - accuracy: 0.8919
    Epoch 217/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0283 - accuracy: 0.8919
    Epoch 218/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0307 - accuracy: 0.8919
    Epoch 219/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0293 - accuracy: 0.8919
    Epoch 220/300
    74/74 [==============================] - 0s 135us/sample - loss: 0.0269 - accuracy: 0.8919
    Epoch 221/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0306 - accuracy: 0.8919
    Epoch 222/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0356 - accuracy: 0.8919
    Epoch 223/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0251 - accuracy: 0.8919
    Epoch 224/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0247 - accuracy: 0.8919
    Epoch 225/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0276 - accuracy: 0.8919
    Epoch 226/300
    74/74 [==============================] - 0s 130us/sample - loss: 0.0312 - accuracy: 0.8919
    Epoch 227/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0299 - accuracy: 0.8919
    Epoch 228/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0282 - accuracy: 0.8919
    Epoch 229/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0289 - accuracy: 0.8919
    Epoch 230/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0291 - accuracy: 0.8919
    Epoch 231/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0266 - accuracy: 0.8919
    Epoch 232/300
    74/74 [==============================] - 0s 129us/sample - loss: 0.0269 - accuracy: 0.8919
    Epoch 233/300
    74/74 [==============================] - 0s 150us/sample - loss: 0.0259 - accuracy: 0.8919
    Epoch 234/300
    74/74 [==============================] - 0s 136us/sample - loss: 0.0308 - accuracy: 0.8919
    Epoch 235/300
    74/74 [==============================] - 0s 185us/sample - loss: 0.0306 - accuracy: 0.8919
    Epoch 236/300
    74/74 [==============================] - 0s 142us/sample - loss: 0.0309 - accuracy: 0.8919
    Epoch 237/300
    74/74 [==============================] - 0s 142us/sample - loss: 0.0306 - accuracy: 0.8919
    Epoch 238/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0299 - accuracy: 0.8919
    Epoch 239/300
    74/74 [==============================] - 0s 132us/sample - loss: 0.0265 - accuracy: 0.8919
    Epoch 240/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0300 - accuracy: 0.8919
    Epoch 241/300
    74/74 [==============================] - 0s 135us/sample - loss: 0.0280 - accuracy: 0.8919
    Epoch 242/300
    74/74 [==============================] - 0s 130us/sample - loss: 0.0277 - accuracy: 0.8919
    Epoch 243/300
    74/74 [==============================] - ETA: 0s - loss: 0.0272 - accuracy: 0.86 - 0s 120us/sample - loss: 0.0327 - accuracy: 0.8919
    Epoch 244/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0320 - accuracy: 0.8919
    Epoch 245/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0308 - accuracy: 0.8919
    Epoch 246/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0283 - accuracy: 0.8919
    Epoch 247/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0294 - accuracy: 0.8919
    Epoch 248/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0271 - accuracy: 0.8919
    Epoch 249/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0289 - accuracy: 0.8919
    Epoch 250/300
    74/74 [==============================] - 0s 129us/sample - loss: 0.0293 - accuracy: 0.8919
    Epoch 251/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0282 - accuracy: 0.8919
    Epoch 252/300
    74/74 [==============================] - 0s 133us/sample - loss: 0.0281 - accuracy: 0.8919
    Epoch 253/300
    74/74 [==============================] - 0s 125us/sample - loss: 0.0271 - accuracy: 0.8919
    Epoch 254/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0275 - accuracy: 0.8919
    Epoch 255/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0294 - accuracy: 0.8919
    Epoch 256/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0303 - accuracy: 0.8919
    Epoch 257/300
    74/74 [==============================] - 0s 119us/sample - loss: 0.0244 - accuracy: 0.8919
    Epoch 258/300
    74/74 [==============================] - 0s 130us/sample - loss: 0.0305 - accuracy: 0.8919
    Epoch 259/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0296 - accuracy: 0.8919
    Epoch 260/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0294 - accuracy: 0.8919
    Epoch 261/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0273 - accuracy: 0.8919
    Epoch 262/300
    74/74 [==============================] - 0s 120us/sample - loss: 0.0314 - accuracy: 0.8919
    Epoch 263/300
    74/74 [==============================] - 0s 129us/sample - loss: 0.0335 - accuracy: 0.8919
    Epoch 264/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0281 - accuracy: 0.8919
    Epoch 265/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0293 - accuracy: 0.8919
    Epoch 266/300
    74/74 [==============================] - ETA: 0s - loss: 0.0364 - accuracy: 0.86 - 0s 123us/sample - loss: 0.0258 - accuracy: 0.8919
    Epoch 267/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0321 - accuracy: 0.8919
    Epoch 268/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0280 - accuracy: 0.8919
    Epoch 269/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0319 - accuracy: 0.8919
    Epoch 270/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0252 - accuracy: 0.8919
    Epoch 271/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0268 - accuracy: 0.8919
    Epoch 272/300
    74/74 [==============================] - 0s 141us/sample - loss: 0.0303 - accuracy: 0.8919
    Epoch 273/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0264 - accuracy: 0.8919
    Epoch 274/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0273 - accuracy: 0.8919
    Epoch 275/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0335 - accuracy: 0.8919
    Epoch 276/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0312 - accuracy: 0.8919
    Epoch 277/300
    74/74 [==============================] - 0s 142us/sample - loss: 0.0307 - accuracy: 0.8919
    Epoch 278/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0271 - accuracy: 0.8919
    Epoch 279/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0254 - accuracy: 0.8919
    Epoch 280/300
    74/74 [==============================] - 0s 126us/sample - loss: 0.0267 - accuracy: 0.8919
    Epoch 281/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0265 - accuracy: 0.8919
    Epoch 282/300
    74/74 [==============================] - 0s 130us/sample - loss: 0.0293 - accuracy: 0.8919
    Epoch 283/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0322 - accuracy: 0.8919
    Epoch 284/300
    74/74 [==============================] - 0s 122us/sample - loss: 0.0256 - accuracy: 0.8919
    Epoch 285/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0296 - accuracy: 0.8919
    Epoch 286/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0291 - accuracy: 0.8919
    Epoch 287/300
    74/74 [==============================] - 0s 118us/sample - loss: 0.0297 - accuracy: 0.8919
    Epoch 288/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0345 - accuracy: 0.8919
    Epoch 289/300
    74/74 [==============================] - 0s 123us/sample - loss: 0.0270 - accuracy: 0.8919
    Epoch 290/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0287 - accuracy: 0.8919
    Epoch 291/300
    74/74 [==============================] - ETA: 0s - loss: 0.0515 - accuracy: 0.73 - 0s 129us/sample - loss: 0.0314 - accuracy: 0.8919
    Epoch 292/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0285 - accuracy: 0.8919
    Epoch 293/300
    74/74 [==============================] - 0s 128us/sample - loss: 0.0303 - accuracy: 0.8919
    Epoch 294/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0291 - accuracy: 0.8919
    Epoch 295/300
    74/74 [==============================] - 0s 120us/sample - loss: 0.0298 - accuracy: 0.8919
    Epoch 296/300
    74/74 [==============================] - 0s 121us/sample - loss: 0.0287 - accuracy: 0.8919
    Epoch 297/300
    74/74 [==============================] - 0s 140us/sample - loss: 0.0298 - accuracy: 0.8919
    Epoch 298/300
    74/74 [==============================] - 0s 119us/sample - loss: 0.0290 - accuracy: 0.8919
    Epoch 299/300
    74/74 [==============================] - 0s 127us/sample - loss: 0.0302 - accuracy: 0.8919
    Epoch 300/300
    74/74 [==============================] - 0s 124us/sample - loss: 0.0310 - accuracy: 0.8919
    Model: "sequential_14"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    dense_34 (Dense)             (None, 128)               4608      
    _________________________________________________________________
    activation_34 (Activation)   (None, 128)               0         
    _________________________________________________________________
    dropout_22 (Dropout)         (None, 128)               0         
    _________________________________________________________________
    dense_35 (Dense)             (None, 1)                 129       
    _________________________________________________________________
    activation_35 (Activation)   (None, 1)                 0         
    =================================================================
    Total params: 4,737
    Trainable params: 4,737
    Non-trainable params: 0
    _________________________________________________________________
    Train on 75 samples
    Epoch 1/300
    75/75 [==============================] - 0s 4ms/sample - loss: 0.1469 - accuracy: 0.4400
    Epoch 2/300
    75/75 [==============================] - 0s 125us/sample - loss: 0.1418 - accuracy: 0.4800
    Epoch 3/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.1320 - accuracy: 0.6533
    Epoch 4/300
    75/75 [==============================] - 0s 120us/sample - loss: 0.1309 - accuracy: 0.6533
    Epoch 5/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.1226 - accuracy: 0.7467
    Epoch 6/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.1153 - accuracy: 0.7733
    Epoch 7/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.1153 - accuracy: 0.7333
    Epoch 8/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.1117 - accuracy: 0.8267
    Epoch 9/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.1069 - accuracy: 0.8267
    Epoch 10/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0965 - accuracy: 0.8267
    Epoch 11/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0983 - accuracy: 0.8000
    Epoch 12/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.1007 - accuracy: 0.8133
    Epoch 13/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.1000 - accuracy: 0.8133
    Epoch 14/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0910 - accuracy: 0.8000
    Epoch 15/300
    75/75 [==============================] - 0s 105us/sample - loss: 0.0849 - accuracy: 0.8533
    Epoch 16/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0848 - accuracy: 0.8533
    Epoch 17/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0836 - accuracy: 0.8133
    Epoch 18/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0772 - accuracy: 0.8533
    Epoch 19/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0832 - accuracy: 0.8533
    Epoch 20/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0777 - accuracy: 0.8533
    Epoch 21/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0716 - accuracy: 0.8533
    Epoch 22/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0711 - accuracy: 0.8533
    Epoch 23/300
    75/75 [==============================] - 0s 119us/sample - loss: 0.0730 - accuracy: 0.8533
    Epoch 24/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.0715 - accuracy: 0.8533
    Epoch 25/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.0703 - accuracy: 0.8533
    Epoch 26/300
    75/75 [==============================] - 0s 120us/sample - loss: 0.0724 - accuracy: 0.8533
    Epoch 27/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0633 - accuracy: 0.8400
    Epoch 28/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0630 - accuracy: 0.8533
    Epoch 29/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0632 - accuracy: 0.8533
    Epoch 30/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0665 - accuracy: 0.8533
    Epoch 31/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0635 - accuracy: 0.8533
    Epoch 32/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0539 - accuracy: 0.8533
    Epoch 33/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0621 - accuracy: 0.8533
    Epoch 34/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0615 - accuracy: 0.8533
    Epoch 35/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0580 - accuracy: 0.8533
    Epoch 36/300
    75/75 [==============================] - 0s 104us/sample - loss: 0.0578 - accuracy: 0.8667
    Epoch 37/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0513 - accuracy: 0.8533
    Epoch 38/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0497 - accuracy: 0.8533
    Epoch 39/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0513 - accuracy: 0.8400
    Epoch 40/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0544 - accuracy: 0.8533
    Epoch 41/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0531 - accuracy: 0.8533
    Epoch 42/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0544 - accuracy: 0.8533
    Epoch 43/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0508 - accuracy: 0.8533
    Epoch 44/300
    75/75 [==============================] - 0s 115us/sample - loss: 0.0519 - accuracy: 0.8533
    Epoch 45/300
    75/75 [==============================] - ETA: 0s - loss: 0.0583 - accuracy: 0.86 - 0s 113us/sample - loss: 0.0492 - accuracy: 0.8533
    Epoch 46/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0501 - accuracy: 0.8533
    Epoch 47/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0535 - accuracy: 0.8533
    Epoch 48/300
    75/75 [==============================] - 0s 105us/sample - loss: 0.0510 - accuracy: 0.8533
    Epoch 49/300
    75/75 [==============================] - 0s 115us/sample - loss: 0.0465 - accuracy: 0.8533
    Epoch 50/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0525 - accuracy: 0.8533
    Epoch 51/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0498 - accuracy: 0.8533
    Epoch 52/300
    75/75 [==============================] - 0s 125us/sample - loss: 0.0471 - accuracy: 0.8533
    Epoch 53/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0505 - accuracy: 0.8533
    Epoch 54/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0447 - accuracy: 0.8533
    Epoch 55/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0471 - accuracy: 0.8533
    Epoch 56/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0477 - accuracy: 0.8533
    Epoch 57/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0448 - accuracy: 0.8533
    Epoch 58/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0521 - accuracy: 0.8533
    Epoch 59/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0468 - accuracy: 0.8533
    Epoch 60/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0484 - accuracy: 0.8533
    Epoch 61/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0429 - accuracy: 0.8533
    Epoch 62/300
    75/75 [==============================] - 0s 101us/sample - loss: 0.0457 - accuracy: 0.8667
    Epoch 63/300
    75/75 [==============================] - 0s 116us/sample - loss: 0.0435 - accuracy: 0.8533
    Epoch 64/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0454 - accuracy: 0.8533
    Epoch 65/300
    75/75 [==============================] - ETA: 0s - loss: 0.0505 - accuracy: 0.86 - 0s 111us/sample - loss: 0.0430 - accuracy: 0.8533
    Epoch 66/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.0461 - accuracy: 0.8533
    Epoch 67/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0461 - accuracy: 0.8667
    Epoch 68/300
    75/75 [==============================] - 0s 126us/sample - loss: 0.0432 - accuracy: 0.8533
    Epoch 69/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0429 - accuracy: 0.8533
    Epoch 70/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0392 - accuracy: 0.8533
    Epoch 71/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0398 - accuracy: 0.8533
    Epoch 72/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0427 - accuracy: 0.8533
    Epoch 73/300
    75/75 [==============================] - ETA: 0s - loss: 0.0392 - accuracy: 0.86 - 0s 112us/sample - loss: 0.0443 - accuracy: 0.8533
    Epoch 74/300
    75/75 [==============================] - 0s 105us/sample - loss: 0.0473 - accuracy: 0.8533
    Epoch 75/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0483 - accuracy: 0.8533
    Epoch 76/300
    75/75 [==============================] - 0s 105us/sample - loss: 0.0430 - accuracy: 0.8533
    Epoch 77/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.0395 - accuracy: 0.8533
    Epoch 78/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0420 - accuracy: 0.8533
    Epoch 79/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0459 - accuracy: 0.8533
    Epoch 80/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0353 - accuracy: 0.8533
    Epoch 81/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0414 - accuracy: 0.8533
    Epoch 82/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0386 - accuracy: 0.8533
    Epoch 83/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0430 - accuracy: 0.8533
    Epoch 84/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0380 - accuracy: 0.8533
    Epoch 85/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0420 - accuracy: 0.8667
    Epoch 86/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0411 - accuracy: 0.8533
    Epoch 87/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0402 - accuracy: 0.8533
    Epoch 88/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0429 - accuracy: 0.8533
    Epoch 89/300
    75/75 [==============================] - 0s 120us/sample - loss: 0.0406 - accuracy: 0.8533
    Epoch 90/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0457 - accuracy: 0.8533
    Epoch 91/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0404 - accuracy: 0.8667
    Epoch 92/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0460 - accuracy: 0.8533
    Epoch 93/300
    75/75 [==============================] - 0s 105us/sample - loss: 0.0371 - accuracy: 0.8533
    Epoch 94/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.0398 - accuracy: 0.8533
    Epoch 95/300
    75/75 [==============================] - 0s 116us/sample - loss: 0.0460 - accuracy: 0.8533
    Epoch 96/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0404 - accuracy: 0.8533
    Epoch 97/300
    75/75 [==============================] - ETA: 0s - loss: 0.0082 - accuracy: 0.93 - 0s 118us/sample - loss: 0.0384 - accuracy: 0.8533
    Epoch 98/300
    75/75 [==============================] - 0s 116us/sample - loss: 0.0383 - accuracy: 0.8800
    Epoch 99/300
    75/75 [==============================] - ETA: 0s - loss: 0.0358 - accuracy: 0.80 - 0s 113us/sample - loss: 0.0409 - accuracy: 0.8533
    Epoch 100/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0447 - accuracy: 0.8533
    Epoch 101/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0363 - accuracy: 0.8533
    Epoch 102/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0384 - accuracy: 0.8533
    Epoch 103/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0366 - accuracy: 0.8533
    Epoch 104/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0469 - accuracy: 0.8533
    Epoch 105/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0407 - accuracy: 0.8533
    Epoch 106/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0459 - accuracy: 0.8533
    Epoch 107/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0417 - accuracy: 0.8667
    Epoch 108/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0416 - accuracy: 0.8533
    Epoch 109/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0416 - accuracy: 0.8533
    Epoch 110/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0401 - accuracy: 0.8533
    Epoch 111/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0428 - accuracy: 0.8533
    Epoch 112/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0399 - accuracy: 0.8533
    Epoch 113/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0432 - accuracy: 0.8533
    Epoch 114/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0424 - accuracy: 0.8533
    Epoch 115/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.0397 - accuracy: 0.8533
    Epoch 116/300
    75/75 [==============================] - 0s 115us/sample - loss: 0.0448 - accuracy: 0.8533
    Epoch 117/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0385 - accuracy: 0.8533
    Epoch 118/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0398 - accuracy: 0.8400
    Epoch 119/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0365 - accuracy: 0.8533
    Epoch 120/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0394 - accuracy: 0.8533
    Epoch 121/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0422 - accuracy: 0.8533
    Epoch 122/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0375 - accuracy: 0.8533
    Epoch 123/300
    75/75 [==============================] - 0s 116us/sample - loss: 0.0432 - accuracy: 0.8533
    Epoch 124/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0401 - accuracy: 0.8533
    Epoch 125/300
    75/75 [==============================] - 0s 115us/sample - loss: 0.0428 - accuracy: 0.8533
    Epoch 126/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0382 - accuracy: 0.8533
    Epoch 127/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0458 - accuracy: 0.8533
    Epoch 128/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0391 - accuracy: 0.8533
    Epoch 129/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0393 - accuracy: 0.8533
    Epoch 130/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0388 - accuracy: 0.8533
    Epoch 131/300
    75/75 [==============================] - 0s 105us/sample - loss: 0.0349 - accuracy: 0.8533
    Epoch 132/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0389 - accuracy: 0.8533
    Epoch 133/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0388 - accuracy: 0.8533
    Epoch 134/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0388 - accuracy: 0.8533
    Epoch 135/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0401 - accuracy: 0.8533
    Epoch 136/300
    75/75 [==============================] - 0s 117us/sample - loss: 0.0352 - accuracy: 0.8533
    Epoch 137/300
    75/75 [==============================] - 0s 116us/sample - loss: 0.0379 - accuracy: 0.8533
    Epoch 138/300
    75/75 [==============================] - 0s 104us/sample - loss: 0.0427 - accuracy: 0.8533
    Epoch 139/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0415 - accuracy: 0.8533
    Epoch 140/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0362 - accuracy: 0.8533
    Epoch 141/300
    75/75 [==============================] - ETA: 0s - loss: 0.0436 - accuracy: 0.93 - 0s 111us/sample - loss: 0.0397 - accuracy: 0.8533
    Epoch 142/300
    75/75 [==============================] - 0s 117us/sample - loss: 0.0377 - accuracy: 0.8533
    Epoch 143/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0393 - accuracy: 0.8533
    Epoch 144/300
    75/75 [==============================] - 0s 116us/sample - loss: 0.0389 - accuracy: 0.8533
    Epoch 145/300
    75/75 [==============================] - 0s 116us/sample - loss: 0.0358 - accuracy: 0.8533
    Epoch 146/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0394 - accuracy: 0.8533
    Epoch 147/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0407 - accuracy: 0.8533
    Epoch 148/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0354 - accuracy: 0.8533
    Epoch 149/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0344 - accuracy: 0.8533
    Epoch 150/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0364 - accuracy: 0.8533
    Epoch 151/300
    75/75 [==============================] - 0s 117us/sample - loss: 0.0396 - accuracy: 0.8533
    Epoch 152/300
    75/75 [==============================] - 0s 102us/sample - loss: 0.0358 - accuracy: 0.8533
    Epoch 153/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0362 - accuracy: 0.8533
    Epoch 154/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0371 - accuracy: 0.8533
    Epoch 155/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0408 - accuracy: 0.8533
    Epoch 156/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0373 - accuracy: 0.8533
    Epoch 157/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0417 - accuracy: 0.8533
    Epoch 158/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0387 - accuracy: 0.8533
    Epoch 159/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0447 - accuracy: 0.8533
    Epoch 160/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0387 - accuracy: 0.8533
    Epoch 161/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0398 - accuracy: 0.8533
    Epoch 162/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0343 - accuracy: 0.8533
    Epoch 163/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0414 - accuracy: 0.8533
    Epoch 164/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0365 - accuracy: 0.8533
    Epoch 165/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0427 - accuracy: 0.8533
    Epoch 166/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0388 - accuracy: 0.8533
    Epoch 167/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0398 - accuracy: 0.8533
    Epoch 168/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0389 - accuracy: 0.8533
    Epoch 169/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0394 - accuracy: 0.8533
    Epoch 170/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0409 - accuracy: 0.8533
    Epoch 171/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0382 - accuracy: 0.8533
    Epoch 172/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0399 - accuracy: 0.8533
    Epoch 173/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0378 - accuracy: 0.8533
    Epoch 174/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0369 - accuracy: 0.8533
    Epoch 175/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0401 - accuracy: 0.8533
    Epoch 176/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0370 - accuracy: 0.8667
    Epoch 177/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0371 - accuracy: 0.8533
    Epoch 178/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0351 - accuracy: 0.8533
    Epoch 179/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0401 - accuracy: 0.8533
    Epoch 180/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0387 - accuracy: 0.8533
    Epoch 181/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0366 - accuracy: 0.8533
    Epoch 182/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.0397 - accuracy: 0.8533
    Epoch 183/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0377 - accuracy: 0.8533
    Epoch 184/300
    75/75 [==============================] - 0s 117us/sample - loss: 0.0365 - accuracy: 0.8533
    Epoch 185/300
    75/75 [==============================] - 0s 116us/sample - loss: 0.0412 - accuracy: 0.8533
    Epoch 186/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0408 - accuracy: 0.8533
    Epoch 187/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0374 - accuracy: 0.8533
    Epoch 188/300
    75/75 [==============================] - 0s 104us/sample - loss: 0.0390 - accuracy: 0.8667
    Epoch 189/300
    75/75 [==============================] - ETA: 0s - loss: 0.0380 - accuracy: 0.93 - 0s 112us/sample - loss: 0.0394 - accuracy: 0.8533
    Epoch 190/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0379 - accuracy: 0.8533
    Epoch 191/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0383 - accuracy: 0.8533
    Epoch 192/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0376 - accuracy: 0.8533
    Epoch 193/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0388 - accuracy: 0.8533
    Epoch 194/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0373 - accuracy: 0.8533
    Epoch 195/300
    75/75 [==============================] - 0s 104us/sample - loss: 0.0373 - accuracy: 0.8533
    Epoch 196/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.0354 - accuracy: 0.8533
    Epoch 197/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0357 - accuracy: 0.8533
    Epoch 198/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0434 - accuracy: 0.8533
    Epoch 199/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.0369 - accuracy: 0.8533
    Epoch 200/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0339 - accuracy: 0.8533
    Epoch 201/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0363 - accuracy: 0.8533
    Epoch 202/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0383 - accuracy: 0.8533
    Epoch 203/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0346 - accuracy: 0.8533
    Epoch 204/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0360 - accuracy: 0.8533
    Epoch 205/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0367 - accuracy: 0.8533
    Epoch 206/300
    75/75 [==============================] - 0s 115us/sample - loss: 0.0396 - accuracy: 0.8533
    Epoch 207/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0358 - accuracy: 0.8533
    Epoch 208/300
    75/75 [==============================] - 0s 121us/sample - loss: 0.0409 - accuracy: 0.8533
    Epoch 209/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0427 - accuracy: 0.8533
    Epoch 210/300
    75/75 [==============================] - 0s 115us/sample - loss: 0.0396 - accuracy: 0.8533
    Epoch 211/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0381 - accuracy: 0.8667
    Epoch 212/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0415 - accuracy: 0.8533
    Epoch 213/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0371 - accuracy: 0.8533
    Epoch 214/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0383 - accuracy: 0.8533
    Epoch 215/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0419 - accuracy: 0.8533
    Epoch 216/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.0370 - accuracy: 0.8533
    Epoch 217/300
    75/75 [==============================] - 0s 119us/sample - loss: 0.0328 - accuracy: 0.8533
    Epoch 218/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0357 - accuracy: 0.8533
    Epoch 219/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0347 - accuracy: 0.8533
    Epoch 220/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0365 - accuracy: 0.8533
    Epoch 221/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0388 - accuracy: 0.8533
    Epoch 222/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0378 - accuracy: 0.8533
    Epoch 223/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0388 - accuracy: 0.8533
    Epoch 224/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0353 - accuracy: 0.8533
    Epoch 225/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0394 - accuracy: 0.8533
    Epoch 226/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0377 - accuracy: 0.8533
    Epoch 227/300
    75/75 [==============================] - ETA: 0s - loss: 0.0099 - accuracy: 0.93 - 0s 114us/sample - loss: 0.0383 - accuracy: 0.8533
    Epoch 228/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0407 - accuracy: 0.8533
    Epoch 229/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0414 - accuracy: 0.8533
    Epoch 230/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0363 - accuracy: 0.8533
    Epoch 231/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0399 - accuracy: 0.8533
    Epoch 232/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0403 - accuracy: 0.8533
    Epoch 233/300
    75/75 [==============================] - 0s 104us/sample - loss: 0.0390 - accuracy: 0.8533
    Epoch 234/300
    75/75 [==============================] - 0s 118us/sample - loss: 0.0387 - accuracy: 0.8533
    Epoch 235/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0421 - accuracy: 0.8533
    Epoch 236/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0370 - accuracy: 0.8533
    Epoch 237/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0400 - accuracy: 0.8533
    Epoch 238/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0360 - accuracy: 0.8533
    Epoch 239/300
    75/75 [==============================] - 0s 115us/sample - loss: 0.0406 - accuracy: 0.8533
    Epoch 240/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0393 - accuracy: 0.8533
    Epoch 241/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0392 - accuracy: 0.8533
    Epoch 242/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.0361 - accuracy: 0.8533
    Epoch 243/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0372 - accuracy: 0.8533
    Epoch 244/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0374 - accuracy: 0.8533
    Epoch 245/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0411 - accuracy: 0.8533
    Epoch 246/300
    75/75 [==============================] - 0s 118us/sample - loss: 0.0361 - accuracy: 0.8533
    Epoch 247/300
    75/75 [==============================] - 0s 118us/sample - loss: 0.0358 - accuracy: 0.8533
    Epoch 248/300
    75/75 [==============================] - 0s 122us/sample - loss: 0.0376 - accuracy: 0.8533
    Epoch 249/300
    75/75 [==============================] - 0s 118us/sample - loss: 0.0350 - accuracy: 0.8533
    Epoch 250/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.0382 - accuracy: 0.8533
    Epoch 251/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0345 - accuracy: 0.8533
    Epoch 252/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0369 - accuracy: 0.8533
    Epoch 253/300
    75/75 [==============================] - 0s 114us/sample - loss: 0.0374 - accuracy: 0.8533
    Epoch 254/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0388 - accuracy: 0.8533
    Epoch 255/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0390 - accuracy: 0.8533
    Epoch 256/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0410 - accuracy: 0.8533
    Epoch 257/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0377 - accuracy: 0.8533
    Epoch 258/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0366 - accuracy: 0.8533
    Epoch 259/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0376 - accuracy: 0.8533
    Epoch 260/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.0406 - accuracy: 0.8533
    Epoch 261/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.0366 - accuracy: 0.8533
    Epoch 262/300
    75/75 [==============================] - 0s 104us/sample - loss: 0.0351 - accuracy: 0.8533
    Epoch 263/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0352 - accuracy: 0.8533
    Epoch 264/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0343 - accuracy: 0.8533
    Epoch 265/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0361 - accuracy: 0.8533
    Epoch 266/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0365 - accuracy: 0.8533
    Epoch 267/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0387 - accuracy: 0.8533
    Epoch 268/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0388 - accuracy: 0.8533
    Epoch 269/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0366 - accuracy: 0.8533
    Epoch 270/300
    75/75 [==============================] - 0s 119us/sample - loss: 0.0351 - accuracy: 0.8533
    Epoch 271/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0392 - accuracy: 0.8533
    Epoch 272/300
    75/75 [==============================] - 0s 121us/sample - loss: 0.0377 - accuracy: 0.8533
    Epoch 273/300
    75/75 [==============================] - 0s 116us/sample - loss: 0.0390 - accuracy: 0.8533
    Epoch 274/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0367 - accuracy: 0.8533
    Epoch 275/300
    75/75 [==============================] - 0s 115us/sample - loss: 0.0407 - accuracy: 0.8533
    Epoch 276/300
    75/75 [==============================] - 0s 105us/sample - loss: 0.0405 - accuracy: 0.8533
    Epoch 277/300
    75/75 [==============================] - ETA: 0s - loss: 0.0699 - accuracy: 0.80 - 0s 115us/sample - loss: 0.0391 - accuracy: 0.8533
    Epoch 278/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0367 - accuracy: 0.8533
    Epoch 279/300
    75/75 [==============================] - 0s 113us/sample - loss: 0.0373 - accuracy: 0.8533
    Epoch 280/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0370 - accuracy: 0.8533
    Epoch 281/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0417 - accuracy: 0.8533
    Epoch 282/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0404 - accuracy: 0.8533
    Epoch 283/300
    75/75 [==============================] - 0s 107us/sample - loss: 0.0365 - accuracy: 0.8533
    Epoch 284/300
    75/75 [==============================] - 0s 115us/sample - loss: 0.0386 - accuracy: 0.8533
    Epoch 285/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0365 - accuracy: 0.8533
    Epoch 286/300
    75/75 [==============================] - 0s 117us/sample - loss: 0.0353 - accuracy: 0.8533
    Epoch 287/300
    75/75 [==============================] - 0s 111us/sample - loss: 0.0384 - accuracy: 0.8533
    Epoch 288/300
    75/75 [==============================] - 0s 109us/sample - loss: 0.0373 - accuracy: 0.8533
    Epoch 289/300
    75/75 [==============================] - 0s 117us/sample - loss: 0.0388 - accuracy: 0.8533
    Epoch 290/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0398 - accuracy: 0.8533
    Epoch 291/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0396 - accuracy: 0.8533
    Epoch 292/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0375 - accuracy: 0.8533
    Epoch 293/300
    75/75 [==============================] - 0s 112us/sample - loss: 0.0355 - accuracy: 0.8533
    Epoch 294/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0371 - accuracy: 0.8533
    Epoch 295/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0339 - accuracy: 0.8533
    Epoch 296/300
    75/75 [==============================] - 0s 120us/sample - loss: 0.0371 - accuracy: 0.8533
    Epoch 297/300
    75/75 [==============================] - 0s 108us/sample - loss: 0.0354 - accuracy: 0.8533
    Epoch 298/300
    75/75 [==============================] - 0s 110us/sample - loss: 0.0364 - accuracy: 0.8533
    Epoch 299/300
    75/75 [==============================] - 0s 106us/sample - loss: 0.0370 - accuracy: 0.8533
    Epoch 300/300
    75/75 [==============================] - 0s 103us/sample - loss: 0.0391 - accuracy: 0.8533



```python
print ('Average f1 score', np.mean(test_F1))
print ('Average Run time', np.mean(time_k))
```

    Average f1 score 0.5851851851851851
    Average Run time 3.6827285289764404


#### Building an LSTM Classifier on the sequences for comparison
We built an LSTM Classifier on the sequences to compare the accuracy.


```python
X = darpa_data['seq']
encoded_X = np.ndarray(shape=(len(X),), dtype=list)
for i in range(0,len(X)):
    encoded_X[i]=X.iloc[i].split("~")
```


```python
max_seq_length = np.max(darpa_data['seqlen'])
encoded_X = tf.keras.preprocessing.sequence.pad_sequences(encoded_X, maxlen=max_seq_length)
```


```python
kfold = 3
random_state = 11

test_F1 = np.zeros(kfold)
time_k = np.zeros(kfold)

epochs = 50
batch_size = 15
skf = StratifiedKFold(n_splits=kfold, shuffle=True, random_state=random_state)
k = 0

for train_index, test_index in skf.split(encoded_X, y):
    X_train, X_test = encoded_X[train_index], encoded_X[test_index]
    y_train, y_test = y[train_index], y[test_index]
    
    embedding_vecor_length = 32
    top_words=50
    model = Sequential()
    model.add(Embedding(top_words, embedding_vecor_length, input_length=max_seq_length))
    model.add(LSTM(32))
    model.add(Dense(1))
    model.add(Activation('sigmoid'))
    model.compile(loss='binary_crossentropy', optimizer='adam', metrics=['accuracy'])
    
    model.summary()
    
    start_time = time.time()
    model.fit(X_train, y_train, epochs=epochs, batch_size=batch_size, verbose=1)
    end_time=time.time()
    time_k[k]=end_time-start_time

    y_pred = model.predict_proba(X_test).round().astype(int)
    y_train_pred=model.predict_proba(X_train).round().astype(int)
    test_F1[k]=sklearn.metrics.f1_score(y_test, y_pred)
    k+=1
```

    Model: "sequential_24"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    embedding_9 (Embedding)      (None, 1773, 32)          1600      
    _________________________________________________________________
    lstm_9 (LSTM)                (None, 32)                8320      
    _________________________________________________________________
    dense_44 (Dense)             (None, 1)                 33        
    _________________________________________________________________
    activation_44 (Activation)   (None, 1)                 0         
    =================================================================
    Total params: 9,953
    Trainable params: 9,953
    Non-trainable params: 0
    _________________________________________________________________
    Train on 73 samples
    Epoch 1/50
    73/73 [==============================] - 5s 71ms/sample - loss: 0.6829 - accuracy: 0.8493
    Epoch 2/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.6532 - accuracy: 0.8904
    Epoch 3/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.6164 - accuracy: 0.8904
    Epoch 4/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.5658 - accuracy: 0.8904
    Epoch 5/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.4744 - accuracy: 0.8904
    Epoch 6/50
    73/73 [==============================] - 3s 46ms/sample - loss: 0.3893 - accuracy: 0.8904
    Epoch 7/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3459 - accuracy: 0.8904
    Epoch 8/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3521 - accuracy: 0.8904
    Epoch 9/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3522 - accuracy: 0.8904
    Epoch 10/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3512 - accuracy: 0.8904
    Epoch 11/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3452 - accuracy: 0.8904
    Epoch 12/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3444 - accuracy: 0.8904
    Epoch 13/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3436 - accuracy: 0.8904
    Epoch 14/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3431 - accuracy: 0.8904
    Epoch 15/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3425 - accuracy: 0.8904
    Epoch 16/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3434 - accuracy: 0.8904
    Epoch 17/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3402 - accuracy: 0.8904
    Epoch 18/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3400 - accuracy: 0.8904
    Epoch 19/50
    73/73 [==============================] - 3s 45ms/sample - loss: 0.3378 - accuracy: 0.8904
    Epoch 20/50
    73/73 [==============================] - 3s 46ms/sample - loss: 0.3365 - accuracy: 0.8904
    Epoch 21/50
    73/73 [==============================] - 3s 45ms/sample - loss: 0.3347 - accuracy: 0.8904
    Epoch 22/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3304 - accuracy: 0.8904
    Epoch 23/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3252 - accuracy: 0.8904
    Epoch 24/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3217 - accuracy: 0.8904
    Epoch 25/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.3105 - accuracy: 0.8904
    Epoch 26/50
    73/73 [==============================] - 3s 43ms/sample - loss: 0.2963 - accuracy: 0.8904
    Epoch 27/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.2876 - accuracy: 0.8904
    Epoch 28/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.2561 - accuracy: 0.8904
    Epoch 29/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.2379 - accuracy: 0.8904
    Epoch 30/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.2244 - accuracy: 0.8904
    Epoch 31/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.2243 - accuracy: 0.9041
    Epoch 32/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.2195 - accuracy: 0.9178
    Epoch 33/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.1969 - accuracy: 0.9315
    Epoch 34/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.2076 - accuracy: 0.8767
    Epoch 35/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.2151 - accuracy: 0.8767
    Epoch 36/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.1920 - accuracy: 0.9041
    Epoch 37/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.1963 - accuracy: 0.9041
    Epoch 38/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.2015 - accuracy: 0.9178
    Epoch 39/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.1899 - accuracy: 0.8767
    Epoch 40/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.1780 - accuracy: 0.9178
    Epoch 41/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.1784 - accuracy: 0.9315
    Epoch 42/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.1646 - accuracy: 0.9315
    Epoch 43/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.1633 - accuracy: 0.9315
    Epoch 44/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.1607 - accuracy: 0.9315
    Epoch 45/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.1527 - accuracy: 0.9315
    Epoch 46/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.1644 - accuracy: 0.9315
    Epoch 47/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.1660 - accuracy: 0.9178
    Epoch 48/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.1487 - accuracy: 0.9178
    Epoch 49/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.1992 - accuracy: 0.9315
    Epoch 50/50
    73/73 [==============================] - 3s 44ms/sample - loss: 0.1352 - accuracy: 0.9589
    Model: "sequential_25"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    embedding_10 (Embedding)     (None, 1773, 32)          1600      
    _________________________________________________________________
    lstm_10 (LSTM)               (None, 32)                8320      
    _________________________________________________________________
    dense_45 (Dense)             (None, 1)                 33        
    _________________________________________________________________
    activation_45 (Activation)   (None, 1)                 0         
    =================================================================
    Total params: 9,953
    Trainable params: 9,953
    Non-trainable params: 0
    _________________________________________________________________
    Train on 74 samples
    Epoch 1/50
    74/74 [==============================] - 5s 71ms/sample - loss: 0.6728 - accuracy: 0.8649
    Epoch 2/50
    74/74 [==============================] - 3s 43ms/sample - loss: 0.6344 - accuracy: 0.8649
    Epoch 3/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.5765 - accuracy: 0.8784
    Epoch 4/50
    74/74 [==============================] - 3s 43ms/sample - loss: 0.4936 - accuracy: 0.8784
    Epoch 5/50
    74/74 [==============================] - 3s 45ms/sample - loss: 0.3903 - accuracy: 0.8784
    Epoch 6/50
    74/74 [==============================] - 3s 45ms/sample - loss: 0.3818 - accuracy: 0.8784
    Epoch 7/50
    74/74 [==============================] - 3s 45ms/sample - loss: 0.3885 - accuracy: 0.8784
    Epoch 8/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3802 - accuracy: 0.8784
    Epoch 9/50
    74/74 [==============================] - 3s 43ms/sample - loss: 0.3717 - accuracy: 0.8784
    Epoch 10/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3697 - accuracy: 0.8784
    Epoch 11/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3696 - accuracy: 0.8784
    Epoch 12/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3687 - accuracy: 0.8784
    Epoch 13/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3681 - accuracy: 0.8784
    Epoch 14/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3667 - accuracy: 0.8784
    Epoch 15/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3658 - accuracy: 0.8784
    Epoch 16/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3656 - accuracy: 0.8784
    Epoch 17/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3639 - accuracy: 0.8784
    Epoch 18/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3611 - accuracy: 0.8784
    Epoch 19/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3577 - accuracy: 0.8784
    Epoch 20/50
    74/74 [==============================] - 3s 43ms/sample - loss: 0.3554 - accuracy: 0.8784
    Epoch 21/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3533 - accuracy: 0.8784
    Epoch 22/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3476 - accuracy: 0.8784
    Epoch 23/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3439 - accuracy: 0.8784
    Epoch 24/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3297 - accuracy: 0.8784
    Epoch 25/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.3186 - accuracy: 0.8784
    Epoch 26/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.2966 - accuracy: 0.8784
    Epoch 27/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.2752 - accuracy: 0.8784
    Epoch 28/50
    74/74 [==============================] - 3s 43ms/sample - loss: 0.2624 - accuracy: 0.8784
    Epoch 29/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.2652 - accuracy: 0.8919
    Epoch 30/50
    74/74 [==============================] - 3s 43ms/sample - loss: 0.2547 - accuracy: 0.9054
    Epoch 31/50
    74/74 [==============================] - 3s 43ms/sample - loss: 0.2679 - accuracy: 0.9054
    Epoch 32/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.2486 - accuracy: 0.8919
    Epoch 33/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.2146 - accuracy: 0.9054
    Epoch 34/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.2486 - accuracy: 0.9189
    Epoch 35/50
    74/74 [==============================] - 3s 43ms/sample - loss: 0.2169 - accuracy: 0.9459
    Epoch 36/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.2312 - accuracy: 0.8919
    Epoch 37/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.1977 - accuracy: 0.9459
    Epoch 38/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.2101 - accuracy: 0.9459
    Epoch 39/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.2023 - accuracy: 0.9189
    Epoch 40/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.2046 - accuracy: 0.9324
    Epoch 41/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.1890 - accuracy: 0.9459
    Epoch 42/50
    74/74 [==============================] - 3s 45ms/sample - loss: 0.1811 - accuracy: 0.9459
    Epoch 43/50
    74/74 [==============================] - 3s 45ms/sample - loss: 0.1917 - accuracy: 0.9459
    Epoch 44/50
    74/74 [==============================] - 3s 45ms/sample - loss: 0.1872 - accuracy: 0.9459
    Epoch 45/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.1658 - accuracy: 0.9459
    Epoch 46/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.1739 - accuracy: 0.9459
    Epoch 47/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.1645 - accuracy: 0.9459
    Epoch 48/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.1878 - accuracy: 0.9459
    Epoch 49/50
    74/74 [==============================] - 3s 43ms/sample - loss: 0.1841 - accuracy: 0.9595
    Epoch 50/50
    74/74 [==============================] - 3s 44ms/sample - loss: 0.2039 - accuracy: 0.8919
    Model: "sequential_26"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    embedding_11 (Embedding)     (None, 1773, 32)          1600      
    _________________________________________________________________
    lstm_11 (LSTM)               (None, 32)                8320      
    _________________________________________________________________
    dense_46 (Dense)             (None, 1)                 33        
    _________________________________________________________________
    activation_46 (Activation)   (None, 1)                 0         
    =================================================================
    Total params: 9,953
    Trainable params: 9,953
    Non-trainable params: 0
    _________________________________________________________________
    Train on 75 samples
    Epoch 1/50
    75/75 [==============================] - 5s 66ms/sample - loss: 0.6830 - accuracy: 0.7333
    Epoch 2/50
    75/75 [==============================] - 3s 42ms/sample - loss: 0.6459 - accuracy: 0.8800
    Epoch 3/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.6046 - accuracy: 0.8800
    Epoch 4/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.5368 - accuracy: 0.8800
    Epoch 5/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.4176 - accuracy: 0.8800
    Epoch 6/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3610 - accuracy: 0.8800
    Epoch 7/50
    75/75 [==============================] - 3s 42ms/sample - loss: 0.3993 - accuracy: 0.8800
    Epoch 8/50
    75/75 [==============================] - 3s 42ms/sample - loss: 0.3872 - accuracy: 0.8800
    Epoch 9/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3789 - accuracy: 0.8800
    Epoch 10/50
    75/75 [==============================] - 3s 42ms/sample - loss: 0.3725 - accuracy: 0.8800
    Epoch 11/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3679 - accuracy: 0.8800
    Epoch 12/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3678 - accuracy: 0.8800
    Epoch 13/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3681 - accuracy: 0.8800
    Epoch 14/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3682 - accuracy: 0.8800
    Epoch 15/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3669 - accuracy: 0.8800
    Epoch 16/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3677 - accuracy: 0.8800
    Epoch 17/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3657 - accuracy: 0.8800
    Epoch 18/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3654 - accuracy: 0.8800
    Epoch 19/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3669 - accuracy: 0.8800
    Epoch 20/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3650 - accuracy: 0.8800
    Epoch 21/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3649 - accuracy: 0.8800
    Epoch 22/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3650 - accuracy: 0.8800
    Epoch 23/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3633 - accuracy: 0.8800
    Epoch 24/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3620 - accuracy: 0.8800
    Epoch 25/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3611 - accuracy: 0.8800
    Epoch 26/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3595 - accuracy: 0.8800
    Epoch 27/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3627 - accuracy: 0.8800
    Epoch 28/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3559 - accuracy: 0.8800
    Epoch 29/50
    75/75 [==============================] - 3s 42ms/sample - loss: 0.3512 - accuracy: 0.8800
    Epoch 30/50
    75/75 [==============================] - 3s 42ms/sample - loss: 0.3507 - accuracy: 0.8800
    Epoch 31/50
    75/75 [==============================] - 3s 43ms/sample - loss: 0.3392 - accuracy: 0.8800
    Epoch 32/50
    75/75 [==============================] - 3s 42ms/sample - loss: 0.3340 - accuracy: 0.8800
    Epoch 33/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3115 - accuracy: 0.8800
    Epoch 34/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.2962 - accuracy: 0.8800
    Epoch 35/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.2863 - accuracy: 0.8800
    Epoch 36/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.2715 - accuracy: 0.8800
    Epoch 37/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.2471 - accuracy: 0.8800
    Epoch 38/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.3184 - accuracy: 0.8800
    Epoch 39/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.2991 - accuracy: 0.8800
    Epoch 40/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.2846 - accuracy: 0.8800
    Epoch 41/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.2525 - accuracy: 0.8800
    Epoch 42/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.2494 - accuracy: 0.8800
    Epoch 43/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.2456 - accuracy: 0.8800
    Epoch 44/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.2356 - accuracy: 0.8800
    Epoch 45/50
    75/75 [==============================] - 3s 42ms/sample - loss: 0.2281 - accuracy: 0.9067
    Epoch 46/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.2207 - accuracy: 0.9067
    Epoch 47/50
    75/75 [==============================] - 3s 42ms/sample - loss: 0.2165 - accuracy: 0.8800
    Epoch 48/50
    75/75 [==============================] - 3s 42ms/sample - loss: 0.2136 - accuracy: 0.8933
    Epoch 49/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.2141 - accuracy: 0.9067
    Epoch 50/50
    75/75 [==============================] - 3s 41ms/sample - loss: 0.2053 - accuracy: 0.9067



```python
print ('Average f1 score', np.mean(test_F1))
print ('Average Run time', np.mean(time_k))
```

    Average f1 score 0.5309941520467837
    Average Run time 161.5511829853058


We find that the LSTM classifier gives an F1 score of 0. This may be improved by changing the model. However, we find that the SGT embedding could work with a small and unbalanced data without the need of a complicated classifier model.

LSTM models typically require more data for training and also has significantly more computation time. The LSTM model above took 425.6 secs while the MLP model took just 9.1 secs.


```python

```