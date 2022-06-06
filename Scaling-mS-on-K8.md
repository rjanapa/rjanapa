<b>Scaling Microservices on Kubernetes</b><br>

Measure the performance of microservices to find the ones that are performing poorly, are overworked, or are overloaded at times of peak demand. Figure below shows how one might use the Kubernetes dashboard to understand CPU and memory usage for the microservices.

<img src="https://github.com/rjanapa/rjanapa/blob/main/Scalibility-mS-k8.png" width="500" length="500">

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

Listing below is an extract from Terraform code that provisions a cluster on Azure; we change the vm_size field from Standard_B2ms to Standard_B4ms. This upgrades the size of each VM in our Kubernetes node pool. Instead of two CPUs, we now have four (one for each VM). As part of this change, memory and hard-drive for the VM also increase.

We still only have a single VM in our cluster, but we have increased our VM’s size. In this example, scaling our cluster is as simple as a code change. This is the power of infrastructure-as-code, the technique where we store our infrastructure configuration as code and make changes to our infrastructure by committing code changes that trigger our continuous delivery (CD) pipeline

<img src="https://github.com/rjanapa/rjanapa/blob/main/TF-vertical-scaling-ms-k8.png" width="500" length="500">

reference: https://thenewstack.io/scaling-microservices-on-kubernetes/



