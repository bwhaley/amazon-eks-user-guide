# Installing Calico on Amazon EKS<a name="calico"></a>

[Project Calico](https://www.projectcalico.org/) is a network policy engine for Kubernetes\. With Calico network policy enforcement, you can implement network segmentation and tenant isolation\. This is useful in multi\-tenant environments where you must isolate tenants from each other or when you want to create separate environments for development, staging, and production\. Network policies are similar to AWS security groups in that you can create network ingress and egress rules\. Instead of assigning instances to a security group, you assign network policies to pods using pod selectors and labels\.

**Considerations**
+ Calico is not supported when using Fargate with Amazon EKS\.
+ Calico adds rules to `iptables` on the node that may be higher priority than existing rules that you've already implemented outside of Calico\. Consider adding existing `iptables` rules to your Calico policies to avoid having rules outside of Calico policy overridden by Calico\.
+ If you're using [security groups for pods](security-groups-for-pods.md), traffic flow to pods on branch network interfaces is not subjected to Calico network policy enforcement and is limited to Amazon EC2 security group enforcement only\. Community effort is underway to remove this limitation\.

**Prerequisites**
+ An existing Amazon EKS cluster\. To deploy one, see [Getting started with Amazon EKS](getting-started.md)\.
+ The `kubectl` command line tool installed on your computer or AWS CloudShell\. The version must be the same, or up to two versions later than your cluster version\. To install or upgrade `kubectl`, see [Installing `kubectl`](install-kubectl.md)\.

The following procedure shows you how to install Calico on Linux nodes in your Amazon EKS cluster\. To install Calico on Windows nodes, see [Using Calico on Amazon EKS Windows Containers](http://aws.amazon.com/blogs/containers/open-source-calico-for-windows-containers-on-amazon-eks/)\.

## Install Calico on your Amazon EKS Linux nodes<a name="calico-install"></a>

You can install Calico using the procedure for Helm or manifests\. The manifests are not updated by Amazon EKS, so we recommend using Helm, because the charts are maintained by Tigera\. 

Amazon EKS doesn't test and verify new Tigera operator and Calico functionality on Amazon EKS clusters\. If you encounter issues during installation and usage of Calico, submit issues to [Tigera Operator](https://github.com/tigera/operator) and [Calico Project](https://github.com/projectcalico/calico) directly\. You should always contact Tigera for compatibility of any new Tigera operator and Calico versions before installing them on your cluster\.

------
#### [ Helm ]

**Prerequisite**  
Helm version 3\.0 or later installed on your computer\. To install or upgrade Helm, see [Using Helm with Amazon EKS](helm.md)\.

**To install Calico with Helm**

1. Add Project Calico into your Helm repository\.

   ```
   helm repo add projectcalico https://docs.projectcalico.org/charts
   ```

1. If you already have Calico added, you may want to update it to get the latest released version\.

   ```
   helm repo update                         
   ```

1. Install version 3\.21\.4 or later of the Tigera Calico operator and custom resource definitions\.

   ```
   helm install calico projectcalico/tigera-operator --version v3.21.4
   ```

1. View the resources in the `tigera-operator` namespace\.

   ```
   kubectl get all -n tigera-operator
   ```

   Output

   The values in the `DESIRED` and `READY` columns for the `replicaset` should match\. The values returned for you are different than the *values* in the following output\.

   ```
   NAME                                  READY   STATUS    RESTARTS   AGE
   pod/tigera-operator-c4b9549c7-h4zp5   1/1     Running   0          110m
   ...
   NAME                                        DESIRED   CURRENT   READY   AGE
   replicaset.apps/tigera-operator-c4b9549c7   1         1         1       2m35s
   ```

1. View the resources in the `calico-system` namespace\.

   ```
   kubectl get all -n calico-system
   ```

   Output

   The values in the `DESIRED` and `READY` columns for the `calico-node` `daemonset` should match\. The values in the `DESIRED` and `READY` columns for the two `replicasets` should also match\. The values returned for you are different than the *values* in the following output\.

   ```
   NAME                                          READY   STATUS    RESTARTS   AGE
   pod/calico-kube-controllers-579b45dcf-z5tsf   1/1     Running   0          100m
   pod/calico-node-v9dhf                         1/1     Running   0          100m
   pod/calico-typha-6f9c6786d-f2mc7              1/1     Running   0          100m
   ...
   NAME                                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR              AGE
   daemonset.apps/calico-node              1         1         1       1            1           kubernetes.io/os=linux     100m
   ...
   NAME                                                DESIRED   CURRENT   READY   AGE
   replicaset.apps/calico-kube-controllers-579b45dcf   1         1         1       100m
   replicaset.apps/calico-typha-6f9c6786d              1         1         1       100m
   ```

1. Confirm that the logs for one of your `calico-node`, `calico-typha`, and `tigera-operator` pods don't contain `ERROR`\. Replace the values in the following commands with the values returned in your output for the previous steps\. 

   ```
   kubectl logs tigera-operator-c4b9549c7-h4zp5 -n tigera-operator | grep ERROR
   kubectl logs calico-node-v9dhf -n calico-system | grep ERROR
   kubectl logs calico-typha-6f9c6786d-f2mc7 -n calico-system | grep ERROR
   ```

   If no output is returned from the previous commands, then `ERROR` doesn't exist in your logs and everything should be running correctly\. 

------
#### [ Manifests ]

**Important**  
These charts won't be maintained in the future\. We recommend that you install using Helm instead, because the Helm charts are maintained\. 

**To install Calico using manifests**

1. Apply the Calico manifests to your cluster\. These manifests create a DaemonSet in the `calico-system` namespace\.

   ```
   kubectl apply -f https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/master/config/master/calico-operator.yaml
   kubectl apply -f https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/master/config/master/calico-crs.yaml
   ```

1. View the resources in the `calico-system` namespace\.

   ```
   kubectl get daemonset calico-node --namespace calico-system
   ```

   Output

   The values in the `DESIRED` and `READY` columns should match\. The values returned for you are different than the *values* in the following output\.

   ```
   NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
   calico-node   1         1         1       1            1           kubernetes.io/os=linux   26m
   ```

------

## Stars policy demo<a name="calico-stars-demo"></a>

This section walks through the [Stars policy demo](https://docs.projectcalico.org/v3.1/getting-started/kubernetes/tutorials/stars-policy/) provided by the Project Calico documentation and isn't necessary for Calico functionality on your cluster\. The demo creates a front\-end, back\-end, and client service on your Amazon EKS cluster\. The demo also creates a management graphical user interface that shows the available ingress and egress paths between each service\. We recommend that you complete the demo on a cluster that you don't run production workloads on\. 

Before you create any network policies, all services can communicate bidirectionally\. After you apply the network policies, you can see that the client can only communicate with the front\-end service, and the back\-end only accepts traffic from the front\-end\.

**To run the Stars policy demo**

1. Apply the front\-end, back\-end, client, and management user interface services:

   ```
   kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/00-namespace.yaml
   kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/01-management-ui.yaml
   kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/02-backend.yaml
   kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/03-frontend.yaml
   kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/04-client.yaml
   ```

1. View all pods on the cluster\.

   ```
   kubectl get pods --all-namespaces
   ```

   Output

   In your output, you should see pods in the namespaces shown in the following output\. Your pod *NAMES* and the number of pods in the *READY* column are different than those in the following output\. Don't continue until you see pods with similar names and they all have `Running` in the `STATUS` column\.

   ```
   NAMESPACE         NAME                                       READY   STATUS    RESTARTS   AGE
   ...
   client            client-xlffc                               1/1     Running   0          5m19s
   ...
   management-ui     management-ui-qrb2g                        1/1     Running   0          5m24s
   stars             backend-sz87q                              1/1     Running   0          5m23s
   stars             frontend-cscnf                             1/1     Running   0          5m21s
   ...
   ```

1. To connect to the management user interface, forward your local port 9001 to the `management-ui` service running on your cluster:

   ```
   kubectl port-forward service/management-ui -n management-ui 9001
   ```

1. Open a browser on your local system and point it to [http://localhost:9001/](http://localhost:9001/)\. You should see the management user interface\. The **C** node is the client service, the **F** node is the front\-end service, and the **B** node is the back\-end service\. Each node has full communication access to all other nodes, as indicated by the bold, colored lines\.  
![\[Open network policy\]](http://docs.aws.amazon.com/eks/latest/userguide/images/stars-default.png)

1. Apply the following network policies to isolate the services from each other:

   ```
   kubectl apply -n stars -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/policies/default-deny.yaml
   kubectl apply -n client -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/policies/default-deny.yaml
   ```

1. Refresh your browser\. You see that the management user interface can no longer reach any of the nodes, so they don't show up in the user interface\.

1. Apply the following network policies to allow the management user interface to access the services:

   ```
   kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/policies/allow-ui.yaml
   kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/policies/allow-ui-client.yaml
   ```

1. Refresh your browser\. You see that the management user interface can reach the nodes again, but the nodes cannot communicate with each other\.  
![\[UI access network policy\]](http://docs.aws.amazon.com/eks/latest/userguide/images/stars-no-traffic.png)

1. Apply the following network policy to allow traffic from the front\-end service to the back\-end service:

   ```
   kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/policies/backend-policy.yaml
   ```

1. Refresh your browser\. You see that the front\-end can communicate with the back\-end\.  
![\[Front-end to back-end policy\]](http://docs.aws.amazon.com/eks/latest/userguide/images/stars-front-end-back-end.png)

1. Apply the following network policy to allow traffic from the client to the front\-end service\.

   ```
   kubectl apply -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/policies/frontend-policy.yaml
   ```

1. Refresh your browser\. You see that the client can communicate to the front\-end service\. The front\-end service can still communicate to the back\-end service\.  
![\[Final network policy\]](http://docs.aws.amazon.com/eks/latest/userguide/images/stars-final.png)

1. \(Optional\) When you are done with the demo, you can delete its resources\.

   ```
   kubectl delete -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/04-client.yaml
   kubectl delete -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/03-frontend.yaml
   kubectl delete -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/02-backend.yaml
   kubectl delete -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/01-management-ui.yaml
   kubectl delete -f https://docs.projectcalico.org/v3.5/getting-started/kubernetes/tutorials/stars-policy/manifests/00-namespace.yaml
   ```

   Even after deleting the resources, there can still be `iptables` rules on the nodes that might interfere in unexpected ways with networking in your cluster\. The only sure way to remove Calico is to terminate all of the nodes and recycle them\. To terminate all nodes, either set the Auto Scaling Group desired count to 0, then back up to the desired number, or just terminate the nodes\. If you are unable to recycle the nodes, then see [Disabling and removing Calico Policy](https://github.com/projectcalico/calico/blob/master/calico/hack/remove-calico-policy/remove-policy.md) in the Calico GitHub repository for a last resort procedure\.

## Remove Calico<a name="remove-calico"></a>

Remove Calico using the method that you installed Calico with\.

------
#### [ Helm ]

Remove Calico from your cluster\.

```
helm uninstall calico
```

------
#### [ Manifests ]

Remove Calico from your cluster\.

```
kubectl delete -f https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/master/config/master/calico-crs.yaml
kubectl delete -f https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/master/config/master/calico-operator.yaml
```

------