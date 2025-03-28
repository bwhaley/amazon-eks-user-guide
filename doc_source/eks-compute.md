# Amazon EKS nodes<a name="eks-compute"></a>

Your Amazon EKS cluster can schedule pods on any combination of [Self\-managed nodes](worker.md), Amazon EKS [Managed node groups](managed-node-groups.md), and [AWS Fargate](fargate.md)\. The following table provides several criteria to evaluate when deciding which options best meet your requirements\. We recommend reviewing this page often because the data in this table changes frequently as new capabilities are introduced to Amazon EKS\. To learn more about nodes deployed in your cluster, see [View nodes](view-nodes.md)\.

**Note**  
Bottlerocket has some specific differences from the general information in this table\. For more information, see the Bottlerocket [documentation](https://github.com/bottlerocket-os/bottlerocket/blob/develop/README.md) on GitHub\.


| Criteria | EKS managed node groups | Self managed nodes | AWS Fargate | 
| --- | --- | --- | --- | 
|  Can be deployed to [AWS Outposts](https://docs.aws.amazon.com/outposts/latest/userguide/what-is-outposts.html)  |  No  |  Yes – For more information, see [Amazon EKS on AWS Outposts](eks-on-outposts.md)\.  |  No  | 
|  Can be deployed to [AWS Local Zones](https://aws.amazon.com/about-aws/global-infrastructure/localzones/)  |  No  |  Yes – For more information, see [Amazon EKS on AWS Local Zones](local-zones.md)\.  |  No  | 
|  Can run containers that require Windows  |  No  |  [Yes](windows-support.md) – Your cluster still requires at least one \(two recommended for availability\) Linux node though\.  |  No  | 
|  Can run containers that require Linux  |  Yes  |  Yes  |  Yes  | 
|  Can run workloads that require the Inferentia chip  |  [Yes](inferentia-support.md) – Amazon Linux nodes only  |  [Yes](inferentia-support.md) – Amazon Linux only  |  No  | 
|  Can run workloads that require a GPU  |  [Yes](eks-optimized-ami.md#gpu-ami) – Amazon Linux nodes only  |  [Yes](eks-optimized-ami.md#gpu-ami) – Amazon Linux only  | No | 
|  Can run workloads that require Arm processors  |  [Yes](eks-optimized-ami.md#arm-ami)  |  [Yes](eks-optimized-ami.md#arm-ami)  |  No  | 
| Can run AWS [Bottlerocket](http://aws.amazon.com/bottlerocket/) |  Yes  |  [Yes](launch-node-bottlerocket.md)  |  No  | 
| Pods share a kernel runtime environment with other pods |  Yes – All of your pods on each of your nodes  |  Yes – All of your pods on each of your nodes  |  No – Each pod has a dedicated kernel  | 
|  Pods share CPU, memory, storage, and network resources with other pods\.  |  Yes – Can result in unused resources on each node  |  Yes – Can result in unused resources on each node  |  No – Each pod has dedicated resources and can be sized independently to maximize resource utilization\.   | 
|  Pods can use more hardware and memory than requested in pod specs  |  Yes – If the pod requires more resources than requested, and resources are available on the node, the pod can use additional resources\.  | Yes – If the pod requires more resources than requested, and resources are available on the node, the pod can use additional resources\. |  No – The pod can be re\-deployed using a larger vCPU and memory configuration though\.  | 
|  Must deploy and manage Amazon EC2 instances  |  [Yes](create-managed-node-group.md) – automated through Amazon EKS if you deployed an Amazon EKS optimized AMI\. If you deployed a custom AMI, then you must update the instance manually\.  |  Yes – Manual configuration or using Amazon EKS provided AWS CloudFormation templates to deploy [Linux \(x86\)](launch-workers.md), [Linux \(Arm\)](eks-optimized-ami.md#arm-ami), or [Windows](windows-support.md) nodes\.  |  No  | 
|  Must secure, maintain, and patch the operating system of Amazon EC2 instances  |  Yes  |  Yes  |  No  | 
|  Can provide bootstrap arguments at deployment of a node, such as extra kubelet arguments\.  | Yes – Using a [launch template](launch-templates.md) with a custom AMI |  Yes – For more information, view the [bootstrap script usage information](https://github.com/awslabs/amazon-eks-ami/blob/master/files/bootstrap.sh) on GitHub\.  |  No  | 
| Can assign IP addresses to pods from a different CIDR block than the IP address assigned to the node\. | Yes – Using a [launch template](launch-templates.md) with a custom AMI | Yes, using [CNI custom networking](cni-custom-network.md)\. | No | 
|  Can SSH into node  |  Yes  |  Yes  |  No – There's no node host operating system to SSH to\.  | 
|  Can deploy your own custom AMI to nodes  |  Yes – Using a [launch template](launch-templates.md)  |  Yes  |  No  | 
|  Can deploy your own custom CNI to nodes  |  Yes – Using a [launch template](launch-templates.md) with a custom AMI  |  Yes  |  No  | 
|  Must update node AMI on your own  |  [Yes](update-managed-node-group.md) – If you deployed an Amazon EKS optimized AMI, you're notified in the Amazon EKS console when updates are available\. You can perform the update with one\-click in the console\. If you deployed a custom AMI, you're not notified in the Amazon EKS console when updates are available\. You must perform the update on your own\.  |  [Yes](update-stack.md) – Using tools other than the Amazon EKS console\. This is because self managed nodes can't be managed with the Amazon EKS console\.  |  No  | 
| Must update node Kubernetes version on your own |  [Yes](update-managed-node-group.md) – If you deployed an Amazon EKS optimized AMI, you're notified in the Amazon EKS console when updates are available\. You can perform the update with one click in the console\. If you deployed a custom AMI, you're not notified in the Amazon EKS console when updates are available\. You must perform the update on your own\.  |  [Yes](update-stack.md) – Using tools other than the Amazon EKS console\. This is because self managed nodes can't be managed with the Amazon EKS console\.  | No – You don't manage nodes\. | 
|  Can use Amazon EBS storage with pods  |  [Yes](ebs-csi.md)  |  [Yes](ebs-csi.md)  |  No  | 
|  Can use Amazon EFS storage with pods  |  [Yes](efs-csi.md)  |  [Yes](efs-csi.md)  |  [Yes](efs-csi.md)  | 
|  Can use Amazon FSx for Lustre storage with pods  |  [Yes](fsx-csi.md)  |  [Yes](fsx-csi.md)  |  No  | 
|  Can use Network Load Balancer for services  |  [Yes](network-load-balancing.md)  |  [Yes](network-load-balancing.md)  |  Yes, when using the [Create a network load balancer](network-load-balancing.md#network-load-balancer)  | 
|  Pods can run in a public subnet  |  Yes  |  Yes  |  No  | 
|  Can assign different VPC security groups to individual pods  |  [Yes](security-groups-for-pods.md) – Linux nodes only  | [Yes](security-groups-for-pods.md) – Linux nodes only |  Yes, in 1\.18 or later clusters  | 
|  Can run Kubernetes DaemonSets  |  Yes  |  Yes  |  No  | 
|  Support `HostPort` and `HostNetwork` in the pod manifest  |  Yes  |  Yes  |  No  | 
|  AWS Region availability  |  [All Amazon EKS supported regions](https://docs.aws.amazon.com/general/latest/gr/eks.html)  |  [All Amazon EKS supported regions](https://docs.aws.amazon.com/general/latest/gr/eks.html)  |  [Some Amazon EKS supported regions](fargate.md)  | 
|  Can run containers on EC2 Dedicated Hosts  |  No  |  Yes  |  No  | 
|  Pricing  |  Cost of Amazon EC2 instance that runs multiple pods\. For more information, see [Amazon EC2 pricing](http://aws.amazon.com/ec2/pricing/)\.  |  Cost of Amazon EC2 instance that runs multiple pods\. For more information, see [Amazon EC2 pricing](http://aws.amazon.com/ec2/pricing/)\.  |  Cost of an individual Fargate memory and CPU configuration\. Each pod has its own cost\. For more information, see [AWS Fargate pricing](http://aws.amazon.com/fargate/pricing/)\.  | 