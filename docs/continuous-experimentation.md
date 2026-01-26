# Continuous Experimentation - Redesigned UI

In the design of the model-service, the requests into the model service are currently processed sequentially. Causing the model-service to slower than the fronted, leading to an heavy increase in response time, which in turn leads to a reduction of QoE. 

We hypothesize that horizontally scaling the process-instance count of model-service pods will significantly reduce the <probably average, maybe we can do something fancier> end-to-end latency under significant load. 

The horizontal scaling in this experiment will be done by increasing the count of processes within one pod. In a true deployement, the scaling should be done by scaling the pods instead of the processes. This would amongst other things, allow for horizontal scaling beeond one node. 

When the plan was hypothesized we realized that the frontent was quite limited in the collection of metrics relevant to model-service requests. This because the delay of requests to model-service was aggregated over time, limiting the visualization possibilities. We transitioned into a graph with the 95, 90 and median response at various timepoints. 

There currently was not enough traffic, we created a simple synthetic traffic generator that was ran on the guest os, and increased the requests per second until there was a delay (signfiying significant load).

Since the delay is expected to be load depended, the canary model service and stable model service should experience the same load for simple to interpret results in the dashboard. To achieve this, the load will temporary be split 50/50 between stable and canary by manually pinning the version between the stable and the canary. Note that for the easy of testing Minikube was used instead of a full Kubernetes Cluster.

To run the tests the following is done:
1. An image of model service is build with the threads that model-service uses increased to 16 threads from the default 4 threads.
2. 500 requests are send to the stable model service and canary model service using the pinned versions.
3. In the dasbhoard the results of the different latency counts are compared. 

