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
