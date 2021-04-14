# Poker Hand Prediction
## Data Structures / Algorithms Artifact

### Narrative
PokerHandPrediction is a simple TensorFlow ML Agent that takes 5 playing cards, the two cards dealt and the three community cards on flop, that would be present while playing poker. Then it must predict the final hand you will have after the turn and river, which is a 5-card hand from 7 available cards. I originally created this machine learning agent for my Machine Learning course at Southern New Hampshire University.  

I selected this artifact primarily because I wanted to include machine learning in the portfolio in some way but did not have any that I would be able to complete in time for the assignment deadlines aside from this one. Poker is also the game that I work on at my job, so I have a bit of domain knowledge that I imagine is somewhat applicable to the project. As well, I was not satisfied with the success rate I had achieved originally in the class with my agent and thought it could be improved.  

And as you can see in the figure below, I was able to improve the accuracy to about 65%.  
![figure 1](https://skytech6.github.io/SNHU-ePortfolio/images/finalresults/model_acc.png)

When I originally created this machine learning agent, I was using a 980 Ti as my graphics card and had setup TensorFlow with that gpu configured, in the first week of this course I was waiting on the delivery of a 3070 gpu to replace my fried 980. And as it turns out, the CUDA Toolkit seems to have some problems setting up for the 3070. Thankfully, I do have a strong cpu with plenty of cores to throw at the computing.   

The first thing I did was get a benchmark of the original neural network so I could ensure I had something to compare against to find a higher success rate.  
![figure 2](https://skytech6.github.io/SNHU-ePortfolio/images/original/model_acc.png)
![figure 3](https://skytech6.github.io/SNHU-ePortfolio/images/original/model_loss.png)  

Now what is important to note here is that even though this is 60% accuracy, I realized that I wasn’t actually setting up the model and score correctly.  

```python
history = model.fit(drop_train, hand_train, epochs=100, batch_size=256, verbose=1,
                    validation_data=(drop_test, hand_test), shuffle=True)
```

Should have been

```python
history = model.fit(drop_train, train, epochs=1000, batch_size=256, verbose=1,
                    validation_data=(drop_test, test), shuffle=True)
```

And the same mistake was made on my model evaluation score. The actual success rate of that original agent was only around 43%. I did not notice that mistake until I was already well into refactoring and adjusting the code.  

I believed that I was probably using the wrong type of loss function with Sparse Categorical Crossentroy and wanted to try using a more standard Binary Crossentropy, as it was either pass or fail trailing. This had some success, but ultimately not enough. I then started to look to my optimizer; I was using an SGD optimizer with default learning rate and a bit of applied momentum. I did some research on different keras optimizers and it seemed an almost always good bet would be the Adam optimizer. So I decided to put that in there and see what would happen. I tried with a few different neural network setups until I got this.  

![figure 4](https://skytech6.github.io/SNHU-ePortfolio/images/adam/model_acc.png)  
![figure 5](https://skytech6.github.io/SNHU-ePortfolio/images/adam/model_loss.png)  

Now what I want to note here is that even though that accuracy was only somewhat above 55%, the model loss was consistent and was not dropping to zero after 100 epochs but was still slowly gaining more accuracy. I decided to try a higher number of epochs, 1000 to be exact.  

![figure 6](https://skytech6.github.io/SNHU-ePortfolio/images/finalresults/model_acc.png)  
![figure 7](https://skytech6.github.io/SNHU-ePortfolio/images/finalresults/model_loss.png)

1000 epochs worked! My model had got a tad over 65% on the testing data. I wondered if it could continue to improve if it had an even higher epoch. Now I am aware of overtuning and expected it would, but I set it to 3000 epochs and would scroll through later to see where in the training did overtuning occur. However, this was weird, it never achieved higher than 61%, with more epochs it never even reaches the same level of accuracy as the 1000 epoch of the exact same neural network. I will need to do more research into why this occurs, but now I have hit my peak. 

[Public Github](https://github.com/SkyTech6/PokerHandPrediction)

```python
import pandas
import warnings
from tensorflow import keras
from tensorflow.keras import layers
from tensorflow.keras.optimizers import SGD
from tensorflow.keras import regularizers
import matplotlib.pyplot as plot

warnings.filterwarnings('ignore')

poker_train = pandas.read_csv('poker-hand-training-true.data', header=None)
poker_test = pandas.read_csv('poker-hand-testing.data', header=None)
col = ['Suit of card #1', 'Rank of card #1', 'Suit of card #2', 'Rank of card #2', 'Suit of card #3', 'Rank of card #3',
       'Suit of card #4', 'Rank of card #4', 'Suit of card #5', 'Rank of card 5', 'Poker Hand']

poker_train.columns = col
poker_test.columns = col

hand_train = poker_train['Poker Hand']
hand_test = poker_test['Poker Hand']

train = pandas.get_dummies(hand_train)
test = pandas.get_dummies(hand_test)

drop_train = poker_train.drop('Poker Hand', axis=1)
drop_test = poker_test.drop('Poker Hand', axis=1)

print('Training Set: ', drop_train.shape)
print('Testing Set: ', drop_test.shape)

# Build Neural Network with TensorFlow's Keras support
model = keras.Sequential(
    [
        layers.Dense(5 * 16, activation='relu', input_dim=10, name='dense'),
        layers.Dropout(0.5),
        layers.Dense(5 * 16, activation='relu', name='dense-drop'),
        layers.Dropout(0.2),
        layers.Dense(10, activation='softmax', name='output')
    ]
)

# loss_function = keras.losses.SparseCategoricalCrossentropy()
loss_function = keras.losses.binary_crossentropy
opt = SGD(lr=0.01, momentum=0.9)
adam = keras.optimizers.Adam()
model.compile(loss=loss_function, optimizer=adam, metrics=['accuracy'])

history = model.fit(drop_train, train, epochs=1000, batch_size=256, verbose=1,
                    validation_data=(drop_test, test), shuffle=True)


if __name__ == '__main__':
    score = model.evaluate(drop_test, test, batch_size=256)
    model.summary()
    # plot loss during training
    plot.subplot(211)
    plot.plot(history.history['loss'])
    plot.plot(history.history['val_loss'])
    plot.title('model loss')
    plot.ylabel('loss')
    plot.xlabel('epoc')
    plot.legend(['train', 'test'], loc='upper left')
    plot.show()
    # plot accuracy during training
    plot.subplot(212)
    plot.title('Accuracy')
    plot.plot(history.history['accuracy'])
    plot.plot(history.history['val_accuracy'])
    plot.title('model accuracy')
    plot.ylabel('accuracy')
    plot.legend(['train', 'test'], loc='upper left')
    plot.show()
    ```