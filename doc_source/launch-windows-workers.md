# Launching self\-managed Windows nodes<a name="launch-windows-workers"></a>

This topic helps you to launch an Auto Scaling group of Windows nodes that register with your Amazon EKS cluster\. After the nodes join the cluster, you can deploy Kubernetes applications to them\.

**Important**  
Amazon EKS nodes are standard Amazon EC2 instances, and you are billed for them based on normal Amazon EC2 instance prices\. For more information, see [Amazon EC2 pricing](https://aws.amazon.com/ec2/pricing/)\.

Enable Windows support for your cluster\. We recommend that you review important considerations before you launch a Windows node group\. For more information, see [Enabling Windows support](windows-support.md#enable-windows-support)\. 

You can launch self\-managed Windows nodes with `eksctl` or the AWS Management Console\.

------
#### [ eksctl ]<a name="launch-windows-nodes-eksctl"></a>

**To launch self\-managed Windows nodes using `eksctl`**

This procedure requires that you have installed `eksctl`, and that your `eksctl` version is at least `0.82.0`\. You can check your version with the following command:

```
eksctl version
```

 For instructions on installing or upgrading `eksctl`, see [Installing or upgrading `eksctl`](eksctl.md#installing-eksctl)\.
**Note**  
This procedure only works for clusters that were created with `eksctl`\.

1. \(Optional\) If the **AmazonEKS\_CNI\_Policy** managed IAM policy \(if you have an IPv4 cluster\) or the *AmazonEKS\_CNI\_IPv6\_Policy* \(that you [created yourself](cni-iam-role.md#cni-iam-role-create-ipv6-policy) if you have an IPv6 cluster\) is attached to your [Amazon EKS node IAM role](create-node-role.md), we recommend assigning it to an IAM role that you associate to the Kubernetes `aws-node` service account instead\. For more information, see [Configuring the Amazon VPC CNI plugin to use IAM roles for service accounts](cni-iam-role.md)\.

1. This procedure assumes that you have an existing cluster\. If you don't already have an Amazon EKS cluster and an Amazon Linux 2 node group to add a Windows node group to, we recommend that you follow the [Getting started with Amazon EKS – `eksctl`](getting-started-eksctl.md) guide\. The guide provides a complete walkthrough for how to create an Amazon EKS cluster with Amazon Linux nodes\.

   Create your node group with the following command\. Replace every `example-value` with your own values\.
**Important**  
To deploy a node group to AWS Outposts, AWS Wavelength, or AWS Local Zones subnets, don't pass the AWS Outposts, AWS Wavelength, or AWS Local Zones subnets when you create the cluster\. Create the node group with a config file, specifying the AWS Outposts, AWS Wavelength, or AWS Local Zones subnets\. For more information, see [Create a nodegroup from a config file](https://eksctl.io/usage/managing-nodegroups/#creating-a-nodegroup-from-a-config-file) and [Config file schema](https://eksctl.io/usage/schema/) in the `eksctl` documentation\.

   ```
   eksctl create nodegroup \
     --region region-code \
     --cluster my-cluster \
     --name ng-windows \
     --node-type t2.large \
     --nodes 3 \
     --nodes-min 1 \
     --nodes-max 4 \
     --node-ami-family WindowsServer2019FullContainer
   ```
**Note**  
If nodes fail to join the cluster, see [Nodes fail to join cluster](troubleshooting.md#worker-node-fail) in the Troubleshooting guide\.
To see the available options for `eksctl` commands, enter the following command\.  

     ```
     eksctl command -help
     ```

   The output is as follows\. Several lines are output while the nodes are created\. One of the last lines of output is the following example line\.

   ```
   [✔]  created 1 nodegroup(s) in cluster "my-cluster"
   ```

1. \(Optional\) Deploy a [sample application](sample-deployment.md) to test your cluster and Windows nodes\.

1. \(Optional\) If you plan to assign IAM roles to all of your Kubernetes service accounts so that pods only have the minimum permissions that they need, and no pods in the cluster require access to the Amazon EC2 instance metadata service \(IMDS\) for other reasons, such as retrieving the current AWS Region, then we recommend blocking pod access to IMDS\. For more information, see [Restrict access to the instance profile assigned to the worker node](https://aws.github.io/aws-eks-best-practices/security/docs/iam/#restrict-access-to-the-instance-profile-assigned-to-the-worker-node)\.

------
#### [ AWS Management Console ]

**Prerequisites**
+ An existing Amazon EKS cluster and a Linux node group\. If you don't have these resources, we recommend that you follow one of our [Getting started with Amazon EKS](getting-started.md) guides to create them\. The guides provide a complete walkthrough for how to create an Amazon EKS cluster with Linux nodes\.
+ An existing VPC and security group that meet the requirements for an Amazon EKS cluster\. For more information, see [Cluster VPC and subnet considerations](network_reqs.md) and [Amazon EKS security group considerations](sec-group-reqs.md)\. The [Getting started with Amazon EKS](getting-started.md) guide creates a VPC that meets the requirements\. Alternatively, you can also follow [Creating a VPC for your Amazon EKS cluster](creating-a-vpc.md) to create one manually\.
+ An existing Amazon EKS cluster that uses a VPC and security group that meet the requirements of an Amazon EKS cluster\. For more information, see [Creating an Amazon EKS cluster](create-cluster.md)\. If you have subnets in the AWS Region where you have AWS Outposts, AWS Wavelength, or AWS Local Zones enabled, those subnets must not have been passed in when you created the cluster\.<a name="launch-windows-nodes-console"></a>

**Step 1: To launch self\-managed Windows nodes using the AWS Management Console**

1. Wait for your cluster status to show as `ACTIVE`\. If you launch your nodes before the cluster is active, the nodes fail to register with the cluster and you will have to relaunch them\.

1. Open the AWS CloudFormation console at [https://console\.aws\.amazon\.com/cloudformation](https://console.aws.amazon.com/cloudformation/)

1. Choose **Create stack**\.

1. For **Specify template**, select **Amazon S3 URL**, copy the following URL, paste it into **Amazon S3 URL**, and select **Next** twice\.

   ```
   https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-10-29/amazon-eks-windows-nodegroup.yaml
   ```

1. On the **Quick create stack** page, enter the following parameters accordingly:
   + **Stack name**: Choose a stack name for your AWS CloudFormation stack\. For example, you can call it ***cluster\-name*\-nodes**\.
   + **ClusterName**: Enter the name that you used when you created your Amazon EKS cluster\.
**Important**  
This name must exactly match the name that you used in [Step 1: Create your Amazon EKS cluster](getting-started-console.md#eks-create-cluster)\. Otherwise, your nodes can't join the cluster\.
   + **ClusterControlPlaneSecurityGroup**: Choose the security group from the AWS CloudFormation output that you generated when you created your [VPC](creating-a-vpc.md)\.

     The following steps show one method to retrieve the applicable group\.

     1. Open the Amazon EKS console at [https://console\.aws\.amazon\.com/eks/home\#/clusters](https://console.aws.amazon.com/eks/home#/clusters)\.

     1. Choose the name of the cluster\.

     1. Choose the **Configuration** tab\.

     1. Choose the **Networking** tab\.

     1. Use the **Additional Security Group** value as a reference when selecting from the **ClusterControlPlaneSecurityGroup** drop\-down list\.
   + **NodeGroupName**: Enter a name for your node group\. This name can be used later to identify the Auto Scaling node group that is created for your nodes\.
   + **NodeAutoScalingGroupMinSize**: Enter the minimum number of nodes that your node Auto Scaling group can scale in to\.
   + **NodeAutoScalingGroupDesiredCapacity**: Enter the desired number of nodes to scale to when your stack is created\.
   + **NodeAutoScalingGroupMaxSize**: Enter the maximum number of nodes that your node Auto Scaling group can scale out to\.
   + **NodeInstanceType**: Choose an instance type for your nodes\.
**Note**  
The supported instance types for the latest version of the [Amazon VPC CNI plugin for Kubernetes](https://github.com/aws/amazon-vpc-cni-k8s) are listed in [vpc\_ip\_resource\_limit\.go](https://github.com/aws/amazon-vpc-cni-k8s/blob/release-1.10/pkg/awsutils/vpc_ip_resource_limit.go) on GitHub\. You might need to update your CNI version to use the latest supported instance types\. For more information, see [Updating the Amazon VPC CNI self\-managed add\-on](managing-vpc-cni.md#updating-vpc-cni-add-on)\.
   + **NodeImageIdSSMParam**: Pre\-populated with the Amazon EC2 Systems Manager parameter of the current recommended Amazon EKS optimized Windows Core AMI ID\. To use the full version of Windows, replace `Core` with `Full`\.
   + **NodeImageId**: \(Optional\) If you're using your own custom AMI \(instead of the Amazon EKS optimized AMI\), enter a node AMI ID for your AWS Region\. If you specify a value here, it overrides any values in the **NodeImageIdSSMParam** field\.
   + **NodeVolumeSize**: Specify a root volume size for your nodes, in GiB\.
   + **KeyName**: Enter the name of an Amazon EC2 SSH key pair that you can use to connect using SSH into your nodes with after they launch\. If you don't already have an Amazon EC2 key pair, you can create one in the AWS Management Console\. For more information, see [Amazon EC2 key pairs](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ec2-key-pairs.html) in the *Amazon EC2 User Guide for Windows Instances*\.
**Note**  
If you don't provide a key pair here, the AWS CloudFormation stack creation fails\.
   + **BootstrapArguments**: Specify any optional arguments to pass to the node bootstrap script, such as extra `kubelet` arguments using `-KubeletExtraArgs`\. 
   + **DisableIMDSv1**: Each node supports the Instance Metadata Service Version 1 \(IMDSv1\) and IMDSv2 by default\. However, you can disable IMDSv1\. Select **true** if you don't want any nodes in the node group, or any pods scheduled on the nodes in the node group to use IMDSv1\. For more information about IMDS, see [Configuring the instance metadata service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html)\.
   + **VpcId**: Select the ID for the [VPC](creating-a-vpc.md) that you created\.
   + **NodeSecurityGroups**: Select the security group that was created for your Linux node group when you created your [VPC](creating-a-vpc.md)\. If your Linux nodes have more than one security group attached to them, specify all of them here\. This for, for example, if the Linux node group was created with `eksctl`\.
   + **Subnets**: Choose the subnets that you created\. If you created your VPC using the steps described at [Creating a VPC for your Amazon EKS cluster](creating-a-vpc.md), then specify only the private subnets within the VPC for your nodes to launch into\.
**Important**  
If any of the subnets are public subnets, then they must have the automatic public IP address assignment setting enabled\. If the setting isn't enabled for the public subnet, then any nodes that you deploy to that public subnet won't be assigned a public IP address and won't be able to communicate with the cluster or other AWS services\. If the subnet was deployed before March 26, 2020 using either of the [Amazon EKS AWS CloudFormation VPC templates](creating-a-vpc.md), or by using `eksctl`, then automatic public IP address assignment is disabled for public subnets\. For information about how to enable public IP address assignment for a subnet, see [ Modifying the Public IPv4 Addressing Attribute for Your Subnet](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ip-addressing.html#subnet-public-ip)\. If the node is deployed to a private subnet, then it's able to communicate with the cluster and other AWS services through a NAT gateway\.
If the subnets don't have internet access, then make sure that you're aware of the considerations and extra steps in [Private clusters](private-clusters.md)\.
If you're deploying the nodes in a 1\.18 or earlier cluster, make sure that the subnets you select are tagged with the cluster name\. Replace *my\-cluster* with the name of your cluster and then run the following command to see a list of the subnets currently tagged with your cluster name\.   

       ```
       aws ec2 describe-subnets --filters Name=tag:kubernetes.io/cluster/my-cluster,Values=shared | grep SubnetId
       ```
If the subnet you want to select isn't returned in the output from the previous command, manually add the tag to the subnet\. For more information, see [Subnet tagging](network_reqs.md#vpc-subnet-tagging)\.
If you select AWS Outposts, AWS Wavelength, or AWS Local Zones subnets, then the subnets must not have been passed in when you created the cluster\.

1. Acknowledge that the stack might create IAM resources, and then choose **Create stack**\.

1. When your stack has finished creating, select it in the console and choose **Outputs**\.

1. Record the **NodeInstanceRole** for the node group that was created\. You need this when you configure your Amazon EKS Windows nodes\.

**Step 2: To enable nodes to join your cluster**

1. Download, edit, and apply the AWS IAM Authenticator configuration map\.

   1. Download the configuration map:

      ```
      curl -o aws-auth-cm-windows.yaml https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-10-29/aws-auth-cm-windows.yaml
      ```

   1. Open the file with your favorite text editor\. Replace the *`ARN of instance role (not instance profile) of **Linux** node`* and *`ARN of instance role (not instance profile) of **Windows** node`* snippets with the **NodeInstanceRole** values that you recorded for your Linux and Windows nodes, and save the file\.
**Important**  
Don't modify any other lines in this file\.
Don't use the same IAM role for both Windows and Linux nodes\.

      ```
      apiVersion: v1
      kind: ConfigMap
      metadata:
        name: aws-auth
        namespace: kube-system
      data:
        mapRoles: |
          - rolearn: ARN of instance role (not instance profile) of **Linux** node
            username: system:node:{{EC2PrivateDNSName}}
            groups:
              - system:bootstrappers
              - system:nodes
          - rolearn: ARN of instance role (not instance profile) of **Windows** node
            username: system:node:{{EC2PrivateDNSName}}
            groups:
              - system:bootstrappers
              - system:nodes
              - eks:kube-proxy-windows
      ```

   1. Apply the configuration\. This command may take a few minutes to finish\.

      ```
      kubectl apply -f aws-auth-cm-windows.yaml
      ```
**Note**  
If you receive any authorization or resource type errors, see [Unauthorized or access denied \(`kubectl`\)](troubleshooting.md#unauthorized) in the troubleshooting section\.

      If nodes fail to join the cluster, then see [Nodes fail to join cluster](troubleshooting.md#worker-node-fail) in the Troubleshooting guide\.

1. Watch the status of your nodes and wait for them to reach the `Ready` status\.

   ```
   kubectl get nodes --watch
   ```

1. \(Optional\) Deploy a [sample application](sample-deployment.md) to test your cluster and Windows nodes\.

1. \(Optional\) If the **AmazonEKS\_CNI\_Policy** managed IAM policy \(if you have an IPv4 cluster\) or the *AmazonEKS\_CNI\_IPv6\_Policy* \(that you [created yourself](cni-iam-role.md#cni-iam-role-create-ipv6-policy) if you have an IPv6 cluster\) is attached to your [Amazon EKS node IAM role](create-node-role.md), we recommend assigning it to an IAM role that you associate to the Kubernetes `aws-node` service account instead\. For more information, see [Configuring the Amazon VPC CNI plugin to use IAM roles for service accounts](cni-iam-role.md)\.

1. \(Optional\) If you plan to assign IAM roles to all of your Kubernetes service accounts so that pods only have the minimum permissions that they need, and no pods in the cluster require access to the Amazon EC2 instance metadata service \(IMDS\) for other reasons, such as retrieving the current AWS Region, then we recommend blocking pod access to IMDS\. For more information, see [Restrict access to the instance profile assigned to the worker node](https://aws.github.io/aws-eks-best-practices/security/docs/iam/#restrict-access-to-the-instance-profile-assigned-to-the-worker-node)\.

------