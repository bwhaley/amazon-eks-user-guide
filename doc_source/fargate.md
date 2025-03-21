# AWS Fargate<a name="fargate"></a>

This topic discusses using Amazon EKS to run Kubernetes pods on AWS Fargate\.

AWS Fargate is a technology that provides on\-demand, right\-sized compute capacity for [containers](https://aws.amazon.com/what-are-containers)\. With AWS Fargate, you don't have to provision, configure, or scale groups of virtual machines on your own to run containers\. You also don't need to choose server types, decide when to scale your node groups, or optimize cluster packing\. You can control which pods start on Fargate and how they run with [Fargate profiles](fargate-profile.md)\. Fargate profiles are defined as part of your Amazon EKS cluster\.

Amazon EKS integrates Kubernetes with AWS Fargate by using controllers that are built by AWS using the upstream, extensible model provided by Kubernetes\. These controllers run as part of the Amazon EKS managed Kubernetes control plane and are responsible for scheduling native Kubernetes pods onto Fargate\. The Fargate controllers include a new scheduler that runs alongside the default Kubernetes scheduler in addition to several mutating and validating admission controllers\. When you start a pod that meets the criteria for running on Fargate, the Fargate controllers that are running in the cluster recognize, update, and schedule the pod onto Fargate\.

This topic describes the different components of pods that run on Fargate, and calls out special considerations for using Fargate with Amazon EKS\.

## AWS Fargate considerations<a name="fargate-considerations"></a>

Here are some things to consider about using Fargate on Amazon EKS\.
+ AWS Fargate with Amazon EKS is available in all Amazon EKS Regions except China \(Beijing\), China \(Ningxia\), AWS GovCloud \(US\-East\), and AWS GovCloud \(US\-West\)\.
+ Each pod that runs on Fargate has its own isolation boundary\. They don't share the underlying kernel, CPU resources, memory resources, or elastic network interface with another pod\.
+ Network Load Balancers and Application Load Balancers \(ALBs\) can be used with Fargate with IP targets only\. For more information, see [Create a network load balancer](network-load-balancing.md#network-load-balancer) and [Application load balancing on Amazon EKS](alb-ingress.md)\. 
+ Fargate exposed services only run on target type IP mode, and not on node IP mode\. The recommended way to check the connectivity from a service running on a managed node and a service running on Fargate is to connect via service name\.
+ Pods must match a Fargate profile at the time that they're scheduled to run on Fargate\. Pods that don't match a Fargate profile might be stuck as `Pending`\. If a matching Fargate profile exists, you can delete pending pods that you have created to reschedule them onto Fargate\.
+ You can only use [Security groups for pods](security-groups-for-pods.md) with pods that run on Fargate that are part of a 1\.18 or later cluster\. 
+ Daemonsets aren't supported on Fargate\. If your application requires a daemon, reconfigure that daemon to run as a sidecar container in your pods\.
+ Privileged containers aren't supported on Fargate\.
+ Pods running on Fargate can't specify `HostPort` or `HostNetwork` in the pod manifest\.
+ The default `nofile` and `nproc` soft limit is 1024 and the hard limit is 65535 for Fargate pods\.
+ GPUs aren't currently available on Fargate\.
+ Pods that run on Fargate are only supported on private subnets \(with NAT gateway access to AWS services, but not a direct route to an Internet Gateway\), so your cluster's VPC must have private subnets available\. For clusters without outbound internet access, see [Private clusters](private-clusters.md)\.
+ You can use the [Vertical Pod Autoscaler](vertical-pod-autoscaler.md) to initially right size the CPU and memory for your Fargate pods, and then use the [Horizontal Pod Autoscaler](horizontal-pod-autoscaler.md) to scale those pods\. If you want the Vertical Pod Autoscaler to automatically re\-deploy pods to Fargate with larger CPU and memory combinations, set the mode for the Vertical Pod Autoscaler to either `Auto` or `Recreate` to ensure correct functionality\. For more information, see the [Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler#quick-start) documentation on GitHub\.
+ DNS resolution and DNS hostnames must be enabled for your VPC\. For more information, see [Viewing and updating DNS support for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-dns.html#vpc-dns-updating)\.
+ Amazon EKS Fargate adds defense\-in\-depth for Kubernetes applications by isolating each Pod within a Virtual Machine \(VM\)\. This VM boundary prevents access to host\-based resources used by other Pods in the event of a container escape, which is a common method of attacking containerized applications and gain access to resources outside of the container\.

  Using Amazon EKS doesn't change your responsibilities under the [shared responsibility model](security.md)\. You should carefully consider the configuration of cluster security and governance controls\. The safest way to isolate an application is always to run it in a separate cluster\.
+ Fargate profiles support specifying subnets from VPC secondary CIDR blocks\. You might want to specify a secondary CIDR block\. This is because there's a limited number of IP addresses available in a subnet\. As a result, there's also a limited number of pods that can be created in the cluster\. By using different subnets for pods, you can increase the number of available IP addresses\. For more information, see [Adding IPv4 CIDR blocks to a VPC\.](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#vpc-resize)
+ The Amazon EC2 instance metadata service \(IMDS\) isn't available to pods that are deployed to Fargate nodes\. If you have pods that are deployed to Fargate that need IAM credentials, assign them to your pods using [IAM roles for service accounts](iam-roles-for-service-accounts.md)\. If your pods need access to other information available through IMDS, then you must hard code this information into your pod spec\. This includes the AWS Region or Availability Zone that a pod is deployed to\.
+ You can't deploy Fargate pods to AWS Outposts, AWS Wavelength or AWS Local Zones\.