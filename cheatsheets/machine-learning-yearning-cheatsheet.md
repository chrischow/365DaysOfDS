---
layout: page
title: Machine Learning Yearning Cheatsheet
background: '/img/posts/ml.jpg'
---

## Machine Learning and Scale
* Traditional learning algorithms may or may not do better on small datasets, but certainly don't do as well on large datasets as neural networks (NNs)
* Typically, with more data, the bigger the NN, the better the performance
* The reliable ways to improve an algorithm's performance are usually:
    * Train a bigger network
    * Get more data

## Setting Up Dev and Test Sets

### Definitions
* Training set: Set to run the learning algorithm on
* Dev set (aka hold-out cross validation set): Used to tune parameters, select features, and make other decisions regarding the learning algorithm
* Test set: Used to evaluate the performance of the algorithm, but not to make any decisions regarding what learning algorithm or parameters to use

### Choosing Datasets
* Choose dev and test sets to reflect data you expect to get in the future and want to do well on
* Don't assume your training distribution is the same as your test distribution
* If dev and test sets come from the **same distribution** and the test set result was bad, it means there was overfitting of the dev set
* If dev and test sets come from different distributions, these could have gone wrong:
    * You overfit to the dev set
    * The test set is harder than the dev set (algorithm is ok)
    * The test set is just different from the dev set

### Dataset Sizes
* Key principle: Should be big enough to allow you to detect difference between algorithms that you are comparing
* Dev set:
    * Typical sizes: 1,000 to 10,000
* Test set:
    * For small datasets e.g. 100 to 10,000 samples: 30% of data
    * For big datasets: look at the absolute number and make sure there's enough 

### Establish a Single-Number Evaluation Metric to Optimise
* Allows you to sort models according to their performance on this metric
* If there are multiple related metrics, find a way to combine them e.g. F1 score, weighted average
* Take at most a week to do this + get a test/dev set ready

### Satisficing Metrics
* If there are metrics that can't be combined, define an acceptable level ("good enough") for one metric, and optimise the other metric
* For N total metrics, set N-1 as satisficing metrics

### Iterative Modelling
1. Start off with some **idea**
2. Implement the idea in **code**
3. Carry out an **experiment** - based on what you learned, generate more ideas and go back to step 1

### When to Change Dev/Test Sets and Metrics
1. Actual distribution is different from the dev/test sets: make dev/tests more representative
2. You have overfit to the dev set: get a fresh one
3. The metric is measuring something other than what the project needs to optimise: Change the metric (e.g. adjust penalisation weights)

### Summary from Book
* Choose dev and test sets from a distribution that reflects what data you expect to get in the future and want to do well on. This may not be the same as your training data’s distribution.
* Choose dev and test sets from the same distribution if possible.
* Choose a single-number evaluation metric for your team to optimize. If there are multiple goals that you care about, consider combining them into a single formula (such as averaging multiple error metrics) or defining satisficing and optimizing metrics.
* Machine learning is a highly iterative process: You may try many dozens of ideas before finding one that you’re satisfied with.
* Having dev/test sets and a single-number evaluation metric helps you quickly evaluate algorithms, and therefore iterate faster.
* When starting out on a brand new application, try to establish dev/test sets and a metric quickly, say in less than a week. It might be okay to take longer on mature applications.
* The old heuristic of a 70%/30% train/test split does not apply for problems where you have a lot of data; the dev and test sets can be much less than 30% of the data.
* Your dev set should be large enough to detect meaningful changes in the accuracy of your algorithm, but not necessarily much larger. Your test set should be big enough to give you a confident estimate of the final performance of your system.
* If your dev set and metric are no longer pointing your team in the right direction, quickly change them: (i) If you had overfit the dev set, get more dev set data. (ii) If the actual distribution you care about is different from the dev/test set distribution, get new dev/test set data. (iii) If your metric is no longer measuring what is most important to you, change the metric.

## Basic Error Analysis

### Build Your First System Quickly, Then Iterate
* Build a system quickly and examine how it functions
* This will give clues to show the most promising directions to invest your time

### Error Analysis: Look at Dev Set Examples to Evaluate Ideas
* Error analysis: the process of examining dev set examples that your algorithm misclassified, so that you can understand the underlying causes of the errors
* Estimate the maximum possible improvement from a potential idea (e.g. implementing something to predict a specific class correctly will lower the error by only 5 percentage points)
* Objective is to eliminate categories of errors

### Evaluate Multiple Ideas in Parallel
* Start with enumerating potential ideas in columns
* Put misclassified examples in the rows and see which of them can be corrected using the potential ideas
* Evaluate all these ideas at one shot so you only pass over the set of misclassified examples once
* Add ideas in new columns if you generate them
* Record comments on the possible reasons why the examples were misclassified

### Cleaning Up Mislabelled Dev and Test Set Examples
* In error analysis, also keep track of mislabelled examples
* If the proportion of misclassified examples due to mislabelling was high, it might be worth fixing them
* Otherwise, it's not crucial
* Check the labelling of examples that were labelled correctly as well

### Split Large Dev Set in Two
If dev set is sufficiently large, split it in two:

* Reason is that manual error analysis could lead to overfitting on the misclassified examples inspected
* Blackbox dev set:
    * For tuning parameters
    * Should be about 1,000-10,000 examples
    * If the overall dev set is too small, might have to use the whole dev set as the Eyeball dev set
* Eyeball dev set:
    * For error analysis
    * Will likely overfit to this faster
    * For a problem that even humans don't do well, the Eyeball dev set may not be as helpful and you can omit it
    * Choose size of Eyeball dev set based on the number of mistakes made on it:
        * If error rate is 5%, then an Eyeball set of about 2,000 is required to have 100 misclassified examples
    * With ample data, choose a size based on how much time you have to manually analyse the data
* Guidelines for choosing size of Eyeball set (divide by error rate)
    * ~10 mistakes: Too small, but if the dataset itself it's small, just make do with it
    * ~20 mistakes: Should be able to get a sense of the major error sources
    * ~50 mistakes: Should get a good sense
    * ~100 mistakes: Very good sense
* Eyeball dev set is more important and the overall dev set should be dedicated to this
    * Can use for error analyses, model selection, and hyperparameter tuning
    * Downside is that there is a risk of overfitting

### Book Summary
* When you start a new project, especially if it is in an area in which you are not an expert, it is hard to correctly guess the most promising directions.
* So don’t start off trying to design and build the perfect system. Instead build and train a basic system as quickly as possible—perhaps in a few days. Then use error analysis to help you identify the most promising directions and iteratively improve your algorithm from there.
* Carry out error analysis by manually examining ~100 dev set examples the algorithm misclassifies and counting the major categories of errors. Use this information to prioritize what types of errors to work on fixing.
* Consider splitting the dev set into an Eyeball dev set, which you will manually examine, and a Blackbox dev set, which you will not manually examine. If performance on the Eyeball dev set is much better than the Blackbox dev set, you have overfit the Eyeball dev set and should consider acquiring more data for it.
* The Eyeball dev set should be big enough so that your algorithm misclassifies enough examples for you to analyze. A Blackbox dev set of 1,000-10,000 examples is sufficient for many applications.
* If your dev set is not big enough to split this way, just use the entire dev set as an Eyeball dev set for manual error analysis, model selection, and hyperparameter tuning.

## Bias and Variance

### Introduction
* Bias: Error rate
* Variance: How much worse the algorithm does on the dev (or test) set compared to the training set

### Scenarios

| Bias | Variance | Description  |
| :--: | :------: | :----------- |
| Low  | High     | Overfitting  |
| High | Low      | Underfitting |
| High | High     | Unclear      |
| Low  | Low      | Ideal Situation  |

### Comparing to Optimal Error Rate
* Estimate the optimal error rate (the best you can do) for a problem
    * E.g. Recognising cats = 0% for humans
    * E.g. Recognising speech in noisy environments = Maybe 10% for humans
* Also called Bayes error rate or Bayes rate in statistic
* Bias and variance should be compared to this

### Addressing Bias and Variance
* High bias:
    * Increase the size of your model (e.g. add layers/neurons)
    * Might increase variance and risk of overfitting if regularisation is not being used
    * With regularisation or dropout, performance will stay the same or improve
* High variance: Add data to training set
* Innovative model architectures:
    * Use papers on NNs to get ideas
    * Results are less predictable

### Bias-Variance Tradeoff
* Changes that reduce bias at the cost of variance:
    * Increasing size of model
* Changes that reduce variance but increase bias:
    * Regularisation
* Changes that improve both:
    * Adding input features
    * Proper selection of model architecture (but it's difficult)
* Changes that improve variance:
    * Add training data

### Techniques for Reducing Bias
* Increase the model size e.g. neurons/layers
* Modify input features based on insights from error analysis
    * Perform error analysis on **training data**
    * Select random number of misclassified examples and work through the steps from the Error Analysis chapter
* Reduce or eliminate regularisation
* Modify model architecture

### Techniques for Reducing Variance
* Add more training data
* Add regularisation (also increases bias)
* Add early stopping (based on dev set error)
* Feature selection to decrease number/type of input features
    * Might also increase bias
    * Large reductions would have a bigger effect
    * Less important for deep learning when data is plentiful, more important for smaller datasets
* Decrease model size
    * Could increase bias
    * Not recommended unless you want to reduce computational cost
* Modify input features based on insights from error analysis (also reduces bias)
* Modify model architecture (also reduces bias)

## Learning Curves

### Diagnosing Bias and Variance
* The learning curve:
    * Dev set error:
        * Plots dev set error against number of training examples
        * Run model on different training set sizes
        * As the training set size increases, the dev set error should decrease
        * Add a line for the desired performance: Based on human-level performance and business sense
    * Training error:
        * Training error usually increases as the training set grows: the model finds it harder to memorise the training examples when there is more data
        * Gives confidence to extrapolate the dev set error curve
* Extrapolate from the dev set curve to figure out whether the desired performance can be reached by adding data

High Bias:
* Large gap between training error and desired performance
* Gap between training and dev curves is small, indicating low variance
* Cannot be certain when:
    * Dev is small

High Variance:
* Training error is relatively low
* Dev set error is much higher
* Depending on the slope of the curves, adding data could help

### Learning Curves vs. Dataset Size
* For very small datasets, be wary of:
    * "Bad" or "good" training sets
    * Class imbalance, which leads to unrepresentative training sets
* Solutions to use after you have discerned that the curves are too noisy
    * For the learning curve, create several randomly chosen training sets comprising a low number of examples e.g. 10 of 100 **with replacement**, traing the model, and compute the average training and dev set error
    * Use stratification
* For large datasets, increase the interval for training set sizes to reduce computational cost

## Comparing to Human-Level Performance
* Building an ML system is easier if people can do it well because:
    1. Ease of obtaining data from human labellers: people can provide accurate labels
    2. Error analysis can draw on human intuition:
    3. Use human-level performance to estimate the optimal error rate and set a desired error rate
* Use the highest level of performance that can be feasibly be obtained from a human equivalent
* In error analysis, understanding human-level performance is still important for examples that machines typically get right but the machine didn't

## Training and Testing on Different Distributions

### When to Do It
* All the time if possible
* If true-distribution data is limited, put half into dev/test, and the other half in training

### How to Decide Whether to Use All Your Data
* Traditional learning algorithms: Mixing different distributions would have a risk of worse performance
* Deep learning: Risk is greatly diminished
* Effects of different distributions:
    * It gives the model more examples of what the correct labels are
    * Model expends more capacity learning about properties from the incorrect distribution
* If you have enough computational capacity to build a big enough neural network, having different distributions is not a problem
* Data that is from a completely different distribution that adds no value (e.g. historical documents to recognise cats?!), leave it out

### Weighting Data
* If there are 10x incorrect-distribution examples as there are correct-distribution examples, you should need 10x more computational resources to model both groups of examples
* Weighting examples helps to ensure the model does well on both distributions by assigning a 1/10 weight on the loss from the incorrect-distribution examples
* Impact: don't need to build such a big NN

### Generalising from the Training Set to the Dev Set
* Data mismatch:
    * Model generalises well to new data drawn from the same distribution as the training set
    * If model does well on training dev set (below) but not on the dev set, there is data mismatch
* Split the training set into a **training dev set** which has the same distribution as the training set
* Dev/test sets will continue to reflect the true distribution

### Addressing Data Mismatch
* Try to understand what properties of the data differ between the training and the dev set distributions
* Try to find more training data that better matches the dev set examples

### Artificial Data Synthesis
* Examples:
    * Add noise to speech for speech recognition
    * Add motion blur to images for image recognition
* Beware that synthetic data would appear realistic to a human, but not necessarily to a human e.g. same noise added to speech will cause model to overfit to that noise clip
* Think whether the synthesised data properties are representative of true distribution

## Debugging Inference Algorithms

### The Optimisation Verification Test
*(Doesn't sound very related to simpler ML problems)*

When the model makes incorrect predictions, there are two possibilities for what went wrong:
* Search algorithm problem: the approximate search algorithm failed to find the value of the proposed output that maximises the score of that output
* Objective (scoring function) problem: The function that calculates the score is inaccurate

To diagnose what's wrong, run the Optimisation Verification test by comparing the score of the optimal output against the score of the prediction:
* If the score of the optimal output is higher, it means the search/optimisation algorithm is failing
* If the score of the optimal output is lower, the scoring function is wrong

Thoughts:
* For classification, the correct class will always have a loss of zero
* Therefore, the scoring function will never be wrong, and the fault lies entirely with the optimisation algorithm
* Where does the OV test apply?
    * Problems with more complex scoring functions like speech recognition and translation
    * Custom scoring functions: reinforcement learning

## End-to-End Deep Learning

### Introduction to End-to-End Algorithms
* Learning algorithm maps direct inputs into a desired output
* Intermediate steps (e.g. parsing) are no longer done - the single end-to-end algorithm takes care of everything
* Examples:
    * Sentiment prediction
    * Autonomous driving
    * Speech to text
* Pros:
   * Large dataset: works very well
   * Can learn rich output e.g. text, transcriptions, audio
* Cons:
    * When we don't have much data, hand-engineering features based on knowledge may be more useful
    * Cannot conduct error analysis by parts

### Pipelines
1. Consider how easy it is to collect training data
    * If easy to obtain component data or hard to obtain joint whole-system data: use components
2. Consider how simple the tasks are to be solved by individual components
    * Tasks that are easier to learn: use components
    * Build components that handle simple tasks as opposed to an end-to-end system

## Error Analysis By Parts
* To figure out which parts of the pipeline are problematic
* If there is a human benchmark, check the performance relative to human-level performance for each component and the system as a whole
* Eliminate from error analysis components that are already close
* If errors are ambiguous, manually correct the earlier inputs and test if the components downstream are causing errors
* If all components in the pipeline are performing close to human level, the pipeline is flawed: expand or reconfigure pipeline