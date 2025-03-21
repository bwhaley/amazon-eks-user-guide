# Amazon FSx for Lustre CSI driver<a name="fsx-csi"></a>

The [FSx for Lustre Container Storage Interface \(CSI\) driver](https://github.com/kubernetes-sigs/aws-fsx-csi-driver) provides a CSI interface that allows Amazon EKS clusters to manage the lifecycle of FSx for Lustre file systems\.

This topic shows you how to deploy the FSx for Lustre CSI Driver to your Amazon EKS cluster and verify that it works\. We recommend using version 0\.4\.0 of the driver\.

**Note**  
This driver is supported on Kubernetes version 1\.21 and later Amazon EKS clusters and nodes\. The driver is not supported on Fargate or Arm nodes\. Alpha features of the FSx for Lustre CSI Driver are not supported on Amazon EKS clusters\. The driver is in Beta release\. It is well tested and supported by Amazon EKS for production use\. Support for the driver will not be dropped, though details may change\. If the schema or schematics of the driver changes, instructions for migrating to the next version will be provided\.

For detailed descriptions of the available parameters and complete examples that demonstrate the driver's features, see the [FSx for Lustre Container Storage Interface \(CSI\) driver](https://github.com/kubernetes-sigs/aws-fsx-csi-driver) project on GitHub\.

**Prerequisites**

You must have:
+ Version 2\.4\.9 or later or 1\.22\.30 or later of the AWS CLI installed and configured on your computer or AWS CloudShell\. For more information, see [Installing, updating, and uninstalling the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) and [Quick configuration with `aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config) in the AWS Command Line Interface User Guide\.
+ An existing Amazon EKS cluster\. To deploy one, see [Getting started with Amazon EKS](getting-started.md)\.
+ An existing AWS Identity and Access Management \(IAM\) OpenID Connect \(OIDC\) provider for your cluster\. To determine whether you already have one, or to create one, see [Create an IAM OIDC provider for your cluster](enable-iam-roles-for-service-accounts.md)\.
+ Version 0\.82\.0 or later of the `eksctl` command line tool installed on your computer or AWS CloudShell\. To install or update `eksctl`, see [The `eksctl` command line utility](eksctl.md)\.
+ The `kubectl` command line tool installed on your computer or AWS CloudShell\. The version must be the same, or up to two versions later than your cluster version\. To install or upgrade `kubectl`, see [Installing `kubectl`](install-kubectl.md)\.

**To deploy the FSx for Lustre CSI driver to an Amazon EKS cluster**

1. Create an IAM policy and service account that allows the driver to make calls to AWS APIs on your behalf\.

   1. Copy the following text and save it to a file named `fsx-csi-driver.json`\.

      ```
      {
      
         "Version":"2012-10-17",
         "Statement":[
            {
               "Effect":"Allow",
               "Action":[
                  "iam:CreateServiceLinkedRole",
                  "iam:AttachRolePolicy",
                  "iam:PutRolePolicy"
               ],
               "Resource":"arn:aws:iam::*:role/aws-service-role/s3.data-source.lustre.fsx.amazonaws.com/*"
            },
            {
               "Action":"iam:CreateServiceLinkedRole",
               "Effect":"Allow",
               "Resource":"*",
               "Condition":{
                  "StringLike":{
                     "iam:AWSServiceName":[
                        "fsx.amazonaws.com"
                     ]
                  }
               }
            },
            {
               "Effect":"Allow",
               "Action":[
                  "s3:ListBucket",
                  "fsx:CreateFileSystem",
                  "fsx:DeleteFileSystem",
                  "fsx:DescribeFileSystems",
                  "fsx:TagResource"
               ],
               "Resource":[
                  "*"
               ]
            }
         ]
      }
      ```

   1. Create the policy\. You can replace `Amazon_FSx_Lustre_CSI_Driver` with a different name\.

      ```
      aws iam create-policy \
          --policy-name Amazon_FSx_Lustre_CSI_Driver \
          --policy-document file://fsx-csi-driver.json
      ```

      Take note of the policy Amazon Resource Name \(ARN\) that is returned\.

1. Create a Kubernetes service account for the driver and attach the policy to the service account\. Replacing the ARN of the policy with the ARN returned in the previous step\.

   ```
   eksctl create iamserviceaccount \
       --region region-code \
       --name fsx-csi-controller-sa \
       --namespace kube-system \
       --cluster prod \
       --attach-policy-arn arn:aws:iam::111122223333:policy/Amazon_FSx_Lustre_CSI_Driver \
       --approve
   ```

   Output:

   You'll see several lines of output as the service account is created\. The last line of output is similar to the following example line\.

   ```
   [ℹ]  created serviceaccount "kube-system/fsx-csi-controller-sa"
   ```

   Note the name of the AWS CloudFormation stack that was deployed\. In the example output above, the stack is named `eksctl-prod-addon-iamserviceaccount-kube-system-fsx-csi-controller-sa`\.

1. Note the **Role ARN** for the role that was created\.

   1. Open the AWS CloudFormation console at [https://console\.aws\.amazon\.com/cloudformation](https://console.aws.amazon.com/cloudformation/)\.

   1. Ensure that the console is set to the AWS Region that you created your IAM role in and then select **Stacks**\.

   1. Select the stack named `eksctl-prod-addon-iamserviceaccount-kube-system-fsx-csi-controller-sa`\.

   1. Select the **Outputs** tab\. The **Role ARN** is listed on the **Output\(1\)** page\.

1. Deploy the driver with the following command\.
**Note**  
To see or download the `yaml` file manually, you can find it on the [aws\-fsx\-csi\-driver Github](https://github.com/kubernetes-sigs/aws-fsx-csi-driver/tree/master/deploy/kubernetes/overlays/stable)\.

   ```
   kubectl apply -k "github.com/kubernetes-sigs/aws-fsx-csi-driver/deploy/kubernetes/overlays/stable/?ref=master"
   ```

   Output

   ```
   Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
   serviceaccount/fsx-csi-controller-sa configured
   clusterrole.rbac.authorization.k8s.io/fsx-csi-external-provisioner-role created
   clusterrolebinding.rbac.authorization.k8s.io/fsx-csi-external-provisioner-binding created
   deployment.apps/fsx-csi-controller created
   daemonset.apps/fsx-csi-node created
   csidriver.storage.k8s.io/fsx.csi.aws.com created
   ```

1. Patch the driver deployment to add the service account that you created earlier, replacing the ARN with the ARN that you noted\.

   ```
   kubectl annotate serviceaccount -n kube-system fsx-csi-controller-sa \
    eks.amazonaws.com/role-arn=arn:aws:iam::111122223333:role/eksctl-prod-addon-iamserviceaccount-kube-sys-Role1-NPFTLHJ5PJF5 --overwrite=true
   ```

**To deploy a Kubernetes storage class, persistent volume claim, and sample application to verify that the CSI driver is working**

This procedure uses the [Dynamic volume provisioning for Amazon S3 ](https://github.com/kubernetes-sigs/aws-fsx-csi-driver/tree/master/examples/kubernetes/dynamic_provisioning_s3)from the [FSx for Lustre Container Storage Interface \(CSI\) driver](https://github.com/kubernetes-sigs/aws-fsx-csi-driver) GitHub repository to consume a dynamically\-provisioned FSx for Lustre volume\.

1. Create an Amazon S3 bucket and a folder within it named `export` by creating and copying a file to the bucket\.

   ```
   aws s3 mb s3://fsx-csi
   echo test-file >> testfile
   aws s3 cp testfile s3://fsx-csi/export/testfile
   ```

1. Download the `storageclass` manifest with the following command\.

   ```
   curl -o storageclass.yaml https://raw.githubusercontent.com/kubernetes-sigs/aws-fsx-csi-driver/master/examples/kubernetes/dynamic_provisioning_s3/specs/storageclass.yaml
   ```

1. Edit the file and replace every `example-value` with your own values\.

   ```
   parameters:
     subnetId: subnet-056da83524edbe641
     securityGroupIds: sg-086f61ea73388fb6b
     s3ImportPath: s3://ml-training-data-000
     s3ExportPath: s3://ml-training-data-000/export
     deploymentType: SCRATCH_2
   ```
   + **subnetId** – The subnet ID that the Amazon FSx for Lustre file system should be created in\. Amazon FSx for Lustre isn't supported in all Availability Zones\. Open the Amazon FSx for Lustre console at [https://console\.aws\.amazon\.com/fsx/](https://console.aws.amazon.com/fsx/) to confirm that the subnet that you want to use is in a supported Availability Zone\. The subnet can include your nodes, or can be a different subnet or VPC\. If the subnet that you specify isn't the same subnet that you have nodes in, then your VPCs must be [connected](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/amazon-vpc-to-amazon-vpc-connectivity-options.html), and you must ensure that you have the necessary ports open in your security groups\.
   + **securityGroupIds** – The security group ID for your nodes\.
**Note**  
The security groups must allow inbound/outbound access to Lustre ports 988 and 1021–1023\. For more information, see [Lustre Client VPC Security Group Rules](https://docs.aws.amazon.com/fsx/latest/LustreGuide/limit-access-security-groups.html#lustre-client-inbound-outbound-rules) in the Amazon FSx for Lustre User Guide\.
   + **s3ImportPath** – The Amazon Simple Storage Service data repository that you want to copy data from to the persistent volume\. Specify the `fsx-csi` bucket that you created earlier\.
   + **s3ExportPath** – The Amazon S3 data repository that you want to export new or modified files to\. Specify the `fsx-csi/export` folder that you created earlier\.
   + **deploymentType** – The file system deployment type\. Valid values are `SCRATCH_1`, `SCRATCH_2`, and `PERSISTENT_1`\. For more information about deployment types, see [Create your Amazon FSx for Lustre file system](https://docs.aws.amazon.com/fsx/latest/LustreGuide/getting-started-step1.html)\.
**Note**  
The Amazon S3 bucket for `s3ImportPath` and `s3ExportPath` must be the same, otherwise the driver cannot create the FSx for Lustre file system\. The `s3ImportPath` can stand alone\. A random path will be created automatically like `s3://ml-training-data-000/FSxLustre20190308T012310Z`\. The `s3ExportPath` cannot be used without specifying a value for `S3ImportPath`\.

1. Create the `storageclass`\.

   ```
   kubectl apply -f storageclass.yaml
   ```

1. Download the persistent volume claim manifest\.

   ```
   curl -o claim.yaml https://raw.githubusercontent.com/kubernetes-sigs/aws-fsx-csi-driver/master/examples/kubernetes/dynamic_provisioning_s3/specs/claim.yaml
   ```

1. \(Optional\) Edit the `claim.yaml` file\. Change *1200Gi* to one of the increment values listed below, based on your storage requirements and the `deploymentType` that you selected in a previous step\.

   ```
   storage: 1200Gi
   ```
   + `SCRATCH_2` and `PERSISTENT` – 1\.2 TiB, 2\.4 TiB, or increments of 2\.4 TiB over 2\.4 TiB\.
   + `SCRATCH_1` – 1\.2 TiB, 2\.4 TiB, 3\.6 TiB, or increments of 3\.6 TiB over 3\.6 TiB\.

1. Create the persistent volume claim\.

   ```
   kubectl apply -f claim.yaml
   ```

1. Confirm that the file system is provisioned\.

   ```
   kubectl get pvc
   ```

   Output\.

   ```
   NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
   fsx-claim   Bound    pvc-15dad3c1-2365-11ea-a836-02468c18769e   1200Gi     RWX            fsx-sc         7m37s
   ```
**Note**  
The `STATUS` may show as `Pending` for 5\-10 minutes, before changing to `Bound`\. Don't continue with the next step until the `STATUS` is `Bound`\.

1. Deploy the sample application\.

   ```
   kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-fsx-csi-driver/master/examples/kubernetes/dynamic_provisioning_s3/specs/pod.yaml
   ```

1. Verify that the sample application is running\.

   ```
   kubectl get pods
   ```

   Output

   ```
   NAME      READY   STATUS              RESTARTS   AGE
   fsx-app   1/1     Running             0          8s
   ```

 **Access Amazon S3 files from the FSx for Lustre file system**

If you only want to import data and read it without any modification and creation, then you don't need a value for `s3ExportPath` in your `storageclass.yaml` file\. Verify that data was written to the FSx for Lustre file system by the sample app\.

```
kubectl exec -it fsx-app ls /data
```

Output\.

```
export  out.txt
```

The sample app wrote the `out.txt` file to the file system\.

**Archive files to the `s3ExportPath`**

For new files and modified files, you can use the Lustre user space tool to archive the data back to Amazon S3 using the value that you specified for `s3ExportPath`\.

1. Export the file back to Amazon S3\.

   ```
   kubectl exec -ti fsx-app -- lfs hsm_archive /data/out.txt
   ```
**Note**  
New files aren't synced back to Amazon S3 automatically\. In order to sync files to the `s3ExportPath`, you need to [install the Lustre client](https://docs.aws.amazon.com/fsx/latest/LustreGuide/install-lustre-client.html) in your container image and manually run the `lfs hsm_archive` command\. The container should run in privileged mode with the `CAP_SYS_ADMIN` capability\.
This example uses a lifecycle hook to install the Lustre client for demonstration purpose\. A normal approach is building a container image with the Lustre client\.

1. Confirm that the `out.txt` file was written to the `s3ExportPath` folder in Amazon S3\.

   ```
   aws s3 ls fsx-csi/export/
   ```

   Output

   ```
   2019-12-23 12:11:35       4553 out.txt
   2019-12-23 11:41:21         10 testfile
   ```