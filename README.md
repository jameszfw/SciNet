# SciNet
SciNet is a multi-core based, high performance machine learning framework on top of Spark, supporting both data parallel and model parallel in massive scale.

## Components
Parameter Server, Models, Solver
One System consists of parameter servers, replicated models, and solvers. Each of them are orgnized as a RDD in Spark, consists of multiple partitions.

## Parameter Server
The parameter server is the central repo for the system state. It is orgnized as a RDD with mulitple partitions. How the RDD is partitioned is decided by the partitions used in Model Parallel (discussed later).

It receives parameter update from sub-models, aggregates and feeds updated parameters to sub-models.

In synchronous mode (batch or syncSGD), parameter server receives command from solvers, and calucate the direction, e.g., in L-BGFS/Conjugate Gradient through vector operations, or calculate the gradient by averaging updated from sub-models.

In asynchrounous mode (asyncSGD), parameters receives request form sub-models and replies with updated parameters. The parameter algorithm is AdaGrad or other algorithms.

## Data Parallel
Training data are partitioned into multiple splits, and are trianed independently. Each split corresponds to one RDD in Spark, and calculates the sub-gradient synchronously in Batch or SyncSGD, or asynchronously in AsyncSGD. Correspondingly, it updates/fetch parameter server in sync/async manner.

## Model Parallel
Each model (RDD) can be further partitioned into mulitple splits, and are trained thorugh optimized software pipeline. Each split corresponds to one partition of the given RDD. In this way, each partition hosts one part of the parameters in the whole model. 

Note that parameter server follows exactly the same way as how the model is partitioned in model parallel. By this way, each partition in the model knows exactly which parameter server partition hosts its parameter, and then know exactly where to update and fetch its parameters.

## Function Split
Because one model instance is split into mulitple partitions, to save the memory and network traffic overhead, we use the function split to achieve O(n) complexity with both memory and traffic overhead.

## Pipeline
With the software pipeline, the framework is able to leverage full cluster computation power, and reduce the network latency impact to achieve high performance. The technique is esp. useful in batch training, e.g., L-BFGS, Conjugate Gradient, etc or large minibatch in SGD.

## Solver
In Asynchronous training, the solver is mainly for the purpose of state monitoring and failure recovery.

In Synchronous training, the sovler is the central place to control the training process. It control the sub-gradient calculation in models, aggregate sub-gradients to parameter servers, and compute new directions in parameter servers. 
