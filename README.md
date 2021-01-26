# Optimizing an ML Pipeline in Azure

## Overview
This project is part of the Udacity Azure ML Nanodegree.
In this project, we build and optimize an Azure ML pipeline using the Python SDK and a provided Scikit-learn model.
This model is then compared to an Azure AutoML run.

## Summary
**the problem statement:**
- The dataset provided in this project is UCI Bank Marketing Dataset. This dataset contains information about financial and personal details of customers like job, martial status, education, salary, etc. The y colum or the labels indicates whether the customer subscribed to the fixed term deposit or not. This depicts that this is a classifiacation broblem with two classes or a binary classification problem.

- For this problem we used two approaches HyperDrive and AutoML. Finally we compared the performance.

![Alt text](Screenshots/workflow.png?raw=true "Workflow")

**the solution:**
- Using the HyperDrive i got an accuracy of *0.9144157814871017*
- Using the AutoML the best model is *VotingEnsemble* which gives an accuracy of *0.9171*

## Scikit-learn Pipeline
**The pipeline architecture**
- First we start with the train.py script. this script is used for data preperation. first we get the dataset from the given URL using TabularDatasetFactory. Then we clean the data using clean_data function which uses one hot encoding method. After that splited the data into training and testing set. The script uses logistic regression as a classification alogrithm. Logistic Regression has two arguments regularization strength and no of iterations to converge by default they are defined as 1.0 and 100 respectively.
- Then i created a Standard_D2_V2 Compute Instance named Compute-CPU to run the notebook.

![Alt text](Screenshots/standard_D2_V2.JPG?raw=true "Compute Instance")

- In the notebook we first initializes the exsisting worksape object and created a new experiment to track all the runs in the workspace.
- Then created a Standard_D2_V2 compute cluster with 4 nodes to train the model using the ComputeTarget.
- Next part is the hyperparameter selection and specifying a policy. The two hyperparameters used are C regulizaion strength and max_iter maximum number of iterations to converge for the logistic regression algorithm. I used random parameter sampling to sample over a discrete set of values. the parameter search space used for C is (0.002, 0.02, 0.2, 2.0) and max_iter is (100, 200, 300, 500). I used the Bandit policy. Bandit terminates runs where the primary metric is not within the specified slack factor/slack amount compared to the best performing run.

![Alt text](Screenshots/hyperdrive-configuration.JPG?raw=true "hyperdrive configuration")

- Then i created HyperDriveConfig by passing estimator, policy, hyperparameter sampling and primary metric name on which our model will be measured. I used Accuracy as a primary metric.
- Finally best model is saved which gives the higher accuracy.

![Alt text](Screenshots/bestrun-hyperdrive.JPG?raw=true "bestrun hyperdrive")

![Alt text](Screenshots/hyperdrive-childruns.JPG?raw=true "hyperdrive childruns")

**benefits of the parameter sampler**

The parameter sampler used is Random Sampling. In random sampling, hyperparameter values are randomly selected from among the given search space. Random sampling is chosen because it supports discrete and continuous hyperparameters, and early termination of low-performance runs.

The hyperparameters to be optimized in the Logistic Regression algorithm are --C and --max_iter. Here C value is the Inverse of regularization strength smaller the value stronger the regularization. RandomParameterSampling allows different combinations of both parameters when on the run and from the multiple runs best run will be selected which has the higher accuracy. A continuous range of values is provided for the Random Sampler to choose from. max_iter is the maximum number of iterations it takes for the optimization algorithm to converge. A list of equally spaced options for Max Iterations is provided. From this combination of hyperparameter options, HyperDrive randomly chooses one for each to then find hyperparameter leading to optimal results.

**benefits of the early stopping policy**

Termination policy helps to save time by stopping runs in case of poor performance. Which also helps in saving compute resources. I used the Bandit policy. Bandit terminates runs where the primary metric is not within the specified slack factor/slack amount compared to the best performing run. We can also specify evaluation_interval which is the frequency for applying the policy, and delay_evaluation which delays the first policy evaluation for a specified number of intervals.

## AutoML
**model and hyperparameters generated by AutoML.**

Azure ML creates multiple pipelines in parallel which try on different algorithms and parameters. It iterates through ML algorithms paired with feature selections, and each iteration produces a model with a training score. The higher the score, the better the fitting the model is. The process will stop once the exit criteria defined is hit.

The models attempted via AutoML were RandomForests,BoostedTrees,XGBoost,LightGBM,SGDClassifier,VotingEnsemble etc. AutoML utilized various information preprocessing standardization like Standard Scaling, Min Max Scaling, Sparse Normalizer, MaxAbsScaler etc. The best model selected by the AutoML is *VotingEnsemble* which gives an accuracy of *0.9171*.

## Pipeline comparison

- Both in HyperDrive and AutoML i used accuracy as a primary metric. Hyperdrive uses logistic regression and gave an accuracy of *0.9144157814871017*. While the AutoML chosen best model is *VotingEnsemble* which gives an accuracy of *0.9171*. Both gave around the same results there not that much difference.

- The advantage that the AutoML have is that it tests many configurations and algorithms at the same time, but hyperdrive is limited to the configuration we provide. AutoML pipeline is more easy to setup and lot less configurations are required. One other advantage of AutoML is feature engineering applied automatically.

- In terms of time consumption HyperDrive takes less time while the AutoML takes much larger time to execute.

## Future work

Algorithms other than logistic regression can be used in HyperDrive to improve the results. I used random sampling while we can also use other sampling methods like bayesian and grid can be used. I used Accuracy as a primary metric while other metrics like AUC-weighted might give better results. changing the hyperparameter values of C regulizaion strength and max_iter maximum number of iterations will effect the results.

I used raw uncleaned data in AutoML because AutoML have the capability to tackle this issue. While cleaned data might give better or same results. Howerever using the clean data might also help in reducing the execution time. In AutoML we can also use primary metric other than Accuracy to get better results.

## Proof of cluster clean up

I used ComputeCluster_CPU.delete() at the end of the notbook to delete the compute cluster. Also deleted the Compute Instance.

![Alt text](Screenshots/deleting-Compute-CPU.JPG?raw=true "deleting Compute CPU")
