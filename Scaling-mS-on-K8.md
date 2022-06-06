<b>Scaling Microservices on Kubernetes</b><br>

Measure the performance of microservices to find the ones that are performing poorly, are overworked, or are overloaded at times of peak demand. Figure below shows how one might use the Kubernetes dashboard to understand CPU and memory usage for the microservices.

<img src="https://github.com/rjanapa/rjanapa/blob/main/Scalibility-mS-k8.png" width="500" length="500">

Figure 1: Viewing CPU and memory usage for microservices in the Kubernetes dashboard

<b>Simple techniques for scaling microservices using Kubernetes:</b><br>

1. Vertically scaling the entire cluster<br>
2. Horizontally scaling the entire cluster<br>
3. Horizontally scaling individual microservices<br>
4. Elastically scaling the entire cluster<br>
5. Elastically scaling individual microservices<br>

<b>Vertically Scaling the Cluster</b><br>

As we grow our application, we might come to a point where our cluster generally doesn’t have enough compute, memory or storage to run our application. As we add new microservices (or replicate existing microservices for redundancy), we will eventually max out the nodes in our cluster. (We can monitor this through the k8 dashboard.)

At this point, we must increase the total amount of resources available to our cluster. When scaling microservices on a Kubernetes cluster, we can just as easily make use of either vertical or horizontal scaling.

<img src="https://github.com/rjanapa/rjanapa/blob/main/vertical-scaling-ms-k8.png" width="500" length="500">

Figure 2: Vertically scaling your cluster by increasing the size of the virtual machines (VMs).

Listing below is an extract from Terraform code that provisions a cluster on Azure; we change the vm_size field from Standard_B2ms to Standard_B4ms. This upgrades the size of each VM in our Kubernetes node pool. Instead of two CPUs, we now have four (one for each VM). As part of this change, memory and hard-drive for the VM also increase.

We still only have a single VM in our cluster, but we have increased our VM’s size. In this example, scaling our cluster is as simple as a code change. This is the power of infrastructure-as-code, the technique where we store our infrastructure configuration as code and make changes to our infrastructure by committing code changes that trigger our continuous delivery (CD) pipeline

<img src="https://github.com/rjanapa/rjanapa/blob/main/TF-vertical-scaling-mS-k8.png" width="500" length="500">

Listing 1. Vertically scaling the cluster with Terraform (an extract).

<b>Horizontally Scaling the Cluster</b><br>

In addition to vertically scaling our cluster, we can also scale it horizontally. Our VMs can remain the same size, but we simply add more VMs.

By adding more VMs to our cluster, we spread the load of our application across more computers. Figure 3 illustrates how we can take our cluster from three VMs up to six. The size of each VM remains the same, but we gain more computing power by having more VMs.

<img src="https://github.com/rjanapa/rjanapa/blob/main/horizontal-scaling-ms-k8.png" width="500" length="500">

Figure 3: Horizontally scaling your cluster by increasing the number of VMs.

Listing 2 shows an extract of Terraform code to add more VMs to our node pool. Back in listing 1, we had node_count set to 1, but here we have changed it to 6. Note that we reverted the vm_size field to the smaller size of Standard_B2ms. In this example, we increase the number of VMs, but not their size; although there is nothing stopping us from increasing both the number and the size of our VMs.

Generally, though, we might prefer horizontal scaling because it is less expensive than vertical scaling. That’s because using many smaller VMs is cheaper than using fewer but bigger and higher-priced VMs.

<img src="https://github.com/rjanapa/rjanapa/blob/main/TF-horizontal-scaling-mS-k8.png" width="500" length="500">

Listing 2. Horizontal scaling the cluster with Terraform (an extract).

<b>Horizontally Scaling an Individual Microservice</b><br>

Assuming our cluster is scaled to an adequate size to host all the microservices with good performance, what do we do when individual microservices become overloaded? (This can be monitored in the Kubernetes dashboard.)

Whenever a microservice becomes a performance bottleneck, we can horizontally scale it to distribute its load over multiple instances. This is shown in below figure 4.

<img src="https://github.com/rjanapa/rjanapa/blob/main/horizontal-scaling-a-ms-k8.png" width="500" length="500">

Figure 4: Horizontally scaling a microservice by replicating it.

We are effectively giving more compute, memory and storage to this particular microservice so that it can handle a bigger workload.

Again, we can use code to make this change. We can do this by setting the replicas field in the specification for our Kubernetes deployment or pod as shown in listing 3.

<img src="https://github.com/rjanapa/rjanapa/blob/main/TF-horizontal-scaling-a-ms-k8.png" width="500" length="500">

Listing 3. Horizontally scaling a microservice with Terraform (an extract).

Not only can we scale individual microservices for performance, we can also horizontally scale our microservices for redundancy, creating a more fault-tolerant application. By having multiple instances, there are others available to pick up the load whenever any single instance fails. This allows the failed instance of a microservice to restart and begin working again

<b>Elastic Scaling for the Cluster</b><br>

Moving into more advanced territory, we can now think about elastic scaling. This is a technique where we automatically and dynamically scale our cluster to meet varying levels of demand.

Whenever a demand is low, Kubernetes can automatically deallocate resources that aren’t needed. During high-demand periods, new resources are allocated to meet the increased workload. This generates substantial cost savings because, at any given moment, we only pay for the resources necessary to handle our application’s workload at that time.

We can use elastic scaling at the cluster level to automatically grow our clusters that are nearing their resource limits. Yet again, when using Terraform, this is just a code change. Listing 4 shows how we can enable the Kubernetes autoscaler and set the minimum and maximum size of our node pool.

<img src="https://github.com/rjanapa/rjanapa/blob/main/TF-elastic-scaling-mS-k8.png" width="500" length="500">

Listing 4. Enabling elastic scaling for the cluster with Terraform (an extract)

<b>Elastic Scaling for an Individual Microservice</b><br>

We can also enable elastic scaling at the level of an individual microservice.

Listing 5 is a sample of Terraform code that gives microservices a “burstable” capability. The number of replicas for the microservice is expanded and contracted dynamically to meet the varying workload for the microservice (bursts of activity).

<img src="https://github.com/rjanapa/rjanapa/blob/main/TF-elastic-scaling-a-mS-k8.png" width="500" length="500">

reference: https://thenewstack.io/scaling-microservices-on-kubernetes/



