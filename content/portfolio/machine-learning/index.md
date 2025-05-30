+++
authors = ["Lucien Gheerbrant", "Pierre-Louis Audelan-Ameline"]
title = "Machine Learning & Image Recognition"
description = "Boy, oh boy... Have you ever been struggling deciding if a picture is either that of a car, a horse or a deer?! Who has not? Well this project is for you!"
date = 2023-12-12
[markdown]
numbersections = true
[taxonomies]
tags = ["Machine Learning", "Python", "CNN", "MLP", "TensorFlow"]
[extra]
featured = true
toc = true
katex = true
bibliography = "references.bib"
+++

## Introduction

At the end of 2023, me and my friend, <a class="external" href="https://www.linkedin.com/in/audelan/">Pierre-Louis Audelan-Ameline </a>, had the opportunity to dive into a hands-on Machine Learning (ML) practical session. It was supervised by <a class="external" href="https://pamoellic.github.io/">Pierre-Alain Moellic</a> (great guy). We had to build an Image Recognition (IC) model. IC is one of the key application with ML. You give an image depicting $x$ to a computer, and (hopefully) it guesses that it's $x$ (and not $y$ or $z$). So... How do we get an accurate Car/Deer/Horse recognition software?


## The dataset

### What's in there

The dataset we used is CIFAR-3. It is a modified version of the well-known CIFAR-10 {{ reference(key="krizhevsky2009learning") }} dataset. CIFAR-10 was provided to us by M. Moellic. It is composed of three different classes, as visible on figure 1 below:
<figure>
    {{
        image(
            url='img/classes.webp',
            alt="The first image, placed on the left, shows an orange car. It is labeled 'class 0 (Car)'. The second image, placed on the middle, shows a deer standing in a field. It is labeled 'class 1 (Deer)'. The third image, placed on the right, shows a horse grazing. It is labeled 'class 2 (Horse)'"
        )
    }}
    <figcaption>Fig. 1: Sample images from the CIFAR-10 dataset</figcaption>
</figure>

First and foremost, we need to load the dataset:
```py
import numpy as np

X=np.load('CIFAR-3/X_cifar.npy')
y=np.load('CIFAR-3/Y_cifar.npy')
```
### Do the split!

Nevertheless, let's not forget ML's _golden rule_[^golden-rule]:

> "The test data cannot influence training the model in any way." 

<p style="text-align: right;">
<i>
    - Probably one of the big guys of ML
</i>
</p>

That is to say, if we do not split the database, we will not know if our model works outside of its knowledge base. One of the problem one could run into, if this is not done, could be overfitting.
```py
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(X/255., y, test_size=0.2)

print(str(X_train.shape))
print(str(X_test.shape))
print(str(y_train.shape))
print(str(y_test.shape))
```
{% crt() %}
```
(14400, 32, 32, 3)
(3600, 32, 32, 3)
(14400,)
(3600,)
```
{% end %}

Oh! And let's check if the dataset is balanced. In other words: Is there approximatively the same number of pictures in each class.
```py
car = 0
horse = 0
deer = 0
for i in y_train :
    if i == 0 :
        car += 1
    elif i == 1 :
        deer += 1
    else :
        horse += 1
print("--- Train Set ---")
print("Car : " + str(car))
print("Deer : " + str(deer))
print("Horse : " + str(horse))

car = 0
horse = 0
deer = 0
for i in y_test :
    if i == 0 :
        car += 1
    elif i == 1 :
        deer += 1
    else :
        horse += 1
print("\n--- Test Set ---")
print("Car : " + str(car))
print("Deer : " + str(deer))
print("Horse : " + str(horse))
```
{% crt() %}
```
--- Train Set ---
Car : 4783
Deer : 4807
Horse : 4810

--- Test Set ---
Car : 1217
Deer : 1193
Horse : 1190
```
{% end %}
Yeah! That's pretty balanced. We are almost good to go.

### Dimensionality Reduction with Principal Component Analysis

After loading and balancing the dataset, we moved on to reducing the dimensionality of our data using Principal Component Analysis (PCA). The original dataset had 3072 features (32 $\times$ 32 pixels $\times$ 3 color channels). This is quite high for our poor student laptops. PCA helped us reduce the number of features to a more manageable number while retaining most of the important information. We want to try and keep the most significant details/features, but still reduce dimensionality. That is why we will perform multiple PCAs with different numbers of compenents: To try and find the most balanced PCA transform that doesn’t take away too much information.

<figure>
    {{
        image(
            url='img/var_pca.webp'
        )
    }}
    <figcaption>Fig. 2: Value of the variance ratio depending on the number of components</figcaption>
</figure>

<figure>
    {{
        image(
            url='img/horse_pca.webp'
        )
    }}
    <figcaption>Fig. 3: Effects of the number of components</figcaption>
</figure>

We found that around 200 components captured 95% of the variance. That's what we'll pick.
```py
from sklearn.decomposition import PCA

pca = PCA(n_components=200)
X_train_pca = pca.fit_transform(X_train.reshape(-1, 32*32*3))
X_test_pca = pca.transform(X_test.reshape(-1, 32*32*3))

print(X_train_pca.shape)
print(X_test_pca.shape)
```
{% crt() %}
```
(14400, 200)
(3600, 200)
```
{% end %}
## Supervised Learning Techniques

With the reduced dataset, we applied two supervised learning techniques: Logistic Regression and Gaussian Naive Bayes Classifier. These methods are relatively simple but effective for classification tasks.

### Logistic Regression

We trained a Logistic Regression model on the CIFAR-3-GRAY dataset.
```py
from sklearn.linear_model import LogisticRegression

lr_model = LogisticRegression(max_iter=1000)
lr_model.fit(X_train_pca, y_train)
lr_accuracy = lr_model.score(X_test_pca, y_test)

print(lr_accuracy)
```
{% crt() %}
```
0.595
```
{% end %}
59.5 % accuracy! Quite decent... Especially given the simplicity of the method. 

### Gaussian Naive Bayes Classifier

```py
rom sklearn.naive_bayes import GaussianNB

nbc_model = GaussianNB()
nbc_model.fit(X_train_pca, y_train)
nbc_accuracy = nbc_model.score(X_test_pca, y_test)

print(nbc_accuracy)
```
{% crt() %}
```
0.596
```
{% end %}

Pretty much the same results as Logistic Regression. Pretty nice[^accuracy]!

### Overfitting ? Underfitting ?

If we do not want to gloss over overfitting/underfitting problems, we need to compare the accuracy of the model on the train set with the accuracy on the test set. If the train values are way higher than the test values, the model would be overfitting.
{% crt() %}
    ```
    --- Training Values ---
    Logistic Regression Accuracy: 0.6327083333333333
    Naive Bayes Classifier Accuracy: 0.5833333333333334
    
    --- Testing Values ---
    Logistic Regression Accuracy: 0.6097222222222223
    Naive Bayes Classifier Accuracy: 0.5966666666666667
    ```
{% end %}
The LR training accuracy is slightly above, with a 3.6 % variation.
However, this is low enough to say that there is no overfitting in both
of these models.


## Deep Learning with Multilayer Perceptron

The good stuff! We then explored deep learning using a Multilayer Perceptron (MLP) model. We designed an MLP with input, hidden, and output layers.

### My very first model 🥹

``` py
num_classes = 3

input_layer = Input(shape=(32, 32))

x = Flatten()(input_layer)
x = Dense(128)(x)
x = ReLU()(x)
x = Dropout(0.25)(x)
x = Dense(54)(x)
x = ReLU()(x)
x = Dense(num_classes)(x)
output_layer = Softmax()(x)
model1 = Model(input_layer, output_layer)

model1.compile(
    optimizer=tf.keras.optimizers.Adam(0.001),
    loss=tf.keras.losses.CategoricalCrossentropy(),
    metrics=[tf.keras.metrics.CategoricalAccuracy()],
)

model1.summary()
```
{% crt() %}
```
_________________________________________________________________
Layer (type)                Output Shape              Param #   
=================================================================
input_1 (InputLayer)        [(None, 32, 32)]          0         
                                                                
flatten (Flatten)           (None, 1024)              0         
                                                                
dense (Dense)               (None, 128)               131200    
                                                                
re_lu (ReLU)                (None, 128)               0         
                                                                
dropout (Dropout)           (None, 128)               0         
                                                                
dense_1 (Dense)             (None, 54)                6966      
                                                                
re_lu_1 (ReLU)              (None, 54)                0         
                                                                
dense_2 (Dense)             (None, 3)                 165       
                                                                
softmax (Softmax)           (None, 3)                 0         
                                                                
=================================================================
Total params: 138331 (540.36 KB)
Trainable params: 138331 (540.36 KB)
Non-trainable params: 0 (0.00 Byte)
```
{% end %}

Alright! Let's see how it does...
```
outputs1 = model1.fit(X_train, y_train,epochs=10, batch_size=16,
validation_data=(X_test, y_test))
```

<figure>
    {{
        image(
            url="img/model1.webp"
        )
    }}
    <figcaption>
    Results of our very first model
    </figcaption>
</figure>

We have two hidden layers here. The results are alright compared to the LR and the GNB. Indeed, we
got over the 70 % accuracy bar. But the loss is pretty high. We will need to play around with
the hyperparameters to squeeze out any accuracy.

But that's not all: we can see the model starting to overfit at the 20 epoch mark. After this, the losses of the model on the validation and training set start to greatly divert. The loss is way smaller on the validation set than on the training set. Furthermore, we can see that the model becomes much better at predicting on the training set than on the validation set around the same 20 epoch mark. That's our "sweet spot" then.

### Overfitting, I'm coming for you...

Let's first fight this overtfitting by adding some dropouts. Let's also stop earlier, as the loss and accuracy seem stable pretty early in the epochs.

<figure>
    {{
        image(
            url="img/model2.webp"
        )
    }}
    <figcaption>MLP with dropouts</figcaption>
</figure>

### The more, the merrier right?

Alright, I am a naive engineering student. I believe that if I add more parameters, I have a better model, right?! Let's do this then. We went from 2 layers to 4 layers, with a lot more neurons :

<figure>
    {{
        image(
            url="img/model3.webp"
        )
    }}
    <figcaption>MLP with more layers and neurons</figcaption>
</figure>

Oh. We have almost the same results as in the previous model. This shows that more neurons does not equal more accuracy. Adding complexity is even sometimes harmful to the model.

## Advancing with Convolutional Neural Networks

Finally, we used Convolutional Neural Networks (CNN) for processing color images. The CNN model included convolutional layers, pooling, and dense layers.

### CNN model

``` py
cnn = tf.keras.models.Sequential([
          Conv2D(128, (3, 3), activation='relu', padding='same', input_shape=(32, 32, 3)),
          Conv2D(54, (3, 3), activation='relu'),
          Flatten(),
          Dense(64, activation='relu'),
          Dense(3, activation='softmax')
      ])


cnn.compile(
    optimizer='adam',
    loss=tf.keras.losses.SparseCategoricalCrossentropy(),
    metrics=[tf.keras.metrics.SparseCategoricalAccuracy()],
)

cnn.summary()
```
{% crt() %}
```
_________________________________________________________________
Layer (type)                Output Shape              Param #   
=================================================================
conv2d_14 (Conv2D)          (None, 32, 32, 128)       3584      
                                                                
conv2d_15 (Conv2D)          (None, 30, 30, 54)        62262     
                                                                
flatten_33 (Flatten)        (None, 48600)             0         
                                                                
dense_92 (Dense)            (None, 64)                3110464   
                                                                
dense_93 (Dense)            (None, 3)                 195       
                                                                
=================================================================
Total params: 3176505 (12.12 MB)
Trainable params: 3176505 (12.12 MB)
Non-trainable params: 0 (0.00 Byte)
_________________________________________________________________
```
{% end %}

### Let's run it!

<figure>
    {{
        image(
            url="img/cnn1.webp"
        )
    }}
    <figcaption>Our first CNN</figcaption>
</figure>

The results are way much better than in the MLP and the statistical models. We passed the 80 % accuracy bar, and the loss got below 0.4.

### Making it better

Let's reduce the complexity, but add some pooling layers and some dropouts. We want to diversify the patterns learned by the model. Pooling allows doing spatial filtering mixed with statitics to analyze the data. Let’s also reduce the learning rate, add some convolutional layers and add some dense layers.

<figure>
    {{
        image(
            url="img/cnn2.webp"
        )
    }}
    <figcaption>CNN with a small learning rate (0.001) for its Adam
optimizer</figcaption>
</figure>

We still start to overfit at the 10th epoch mark, but we have now pretty good results, still getting over 85 % accuracy.

## Conclusion

The CNN outperforms the MLP and simpler statistical models significantly, achieving an accuracy exceeding 85 %, while the MLP caps at 75 %. This notable difference in performance can be attributed to several key advantages of CNNs over MLPs.

The convolutional nature of CNNs allows them to excel in finding localized patterns within the data. Unlike MLPs, which attempt to identify patterns across the entire image, CNNs focus on smaller, local regions. This enables them to capture intricate details, making them particularly effective in image-related tasks.

Pooling also enhances the robustness of CNNs by making them less sensitive to spatial variations in the input data. MLPs, on the other hand, are more susceptible to changes in the spatial arrangement of features. Pooling helps CNNs still seek for a pattern, regardless of its size, position and orientation.

CNNs features parameter sharing, which reduces the number of parameters and overall complexity when compared to MLPs. By sharing weights across different regions of the input, CNNs efficiently learn and recognize features in various parts of the image. This not only contributes to a more compact model but also enhances the network's ability to generalize
well to new data. This makes CNN less susceptible to overfitting. With fewer parameters to adjust, CNNs are better equipped to discern meaningful features from the data and are less likely to memorize noise or irrelevant patterns during training than MLPs. This aspect contributes to the CNN's robustness and improved performance on unseen data.

[^golden-rule]: After a web search, I am now aware that there are at least three different golden rules in ML. And that nobody can agree on what is the _golden_ golden rule...
[^accuracy]: Hey, I need 50% to get my engineer degree. If it works for me, it works for the model.