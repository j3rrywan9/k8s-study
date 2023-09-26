# Amazon EKS User Guide

## Amazon EKS clusters

An Amazon EKS cluster consists of two primary components:
* The Amazon EKS control plane
* Amazon EKS nodes that are registered with the control plane

The Amazon EKS control plane consists of control plane nodes that run the Kubernetes software, such as `etcd` and the Kubernetes API server.
The control plane runs in an account managed by AWS, and the Kubernetes API is exposed via the Amazon EKS endpoint associated with your cluster.
Each Amazon EKS cluster control plane is single-tenant and unique, and runs on its own set of Amazon EC2 instances.

All of the data stored by the `etcd` nodes and associated Amazon EBS volumes is encrypted using AWS KMS.
The cluster control plane is provisioned across multiple Availability Zones and fronted by an Elastic Load Balancing Network Load Balancer.
Amazon EKS also provisions elastic network interfaces in your VPC subnets to provide connectivity from the control plane instances to the nodes (for example, to support `kubectl exec logs proxy` data flows).

### Creating an Amazon EKS cluster

When an Amazon EKS cluster is created, the IAM principal that creates the cluster is permanently added to the Kubernetes RBAC authorization table as the administrator.
This principal has `system:masters` permissions.
This principal isn't visible in your cluster configuration.
So, it's important to note the principal that created the cluster and make sure that you never delete it.
Initially, only the IAM principal that created the server can make calls to the Kubernetes API server using `kubectl`.
If you use the console to create the cluster, you must ensure that the same IAM credentials are in the AWS SDK credential chain when you run `kubectl` commands on your cluster.
After your cluster is created, you can grant other IAM principals access to your cluster.

## Amazon EKS nodes

Your Amazon EKS cluster can schedule Pods on any combination of self-managed nodes, Amazon EKS managed node groups, and AWS Fargate.
To learn more about nodes deployed in your cluster, see View Kubernetes resources.

The following table provides several criteria to evaluate when deciding which options best meet your requirements.
This table doesn't include connected nodes that were created outside of Amazon EKS, which can only be viewed.

### Managed node groups

Amazon EKS managed node groups automate the provisioning and lifecycle management of nodes (Amazon EC2 instances) for Amazon EKS Kubernetes clusters.

With Amazon EKS managed node groups, you don't need to separately provision or register the Amazon EC2 instances that provide compute capacity to run your Kubernetes applications.
You can create, automatically update, or terminate nodes for your cluster with a single operation.
Node updates and terminations automatically drain nodes to ensure that your applications stay available.

Every managed node is provisioned as part of an Amazon EC2 Auto Scaling group that's managed for you by Amazon EKS.
Every resource including the instances and Auto Scaling groups runs within your AWS account.
Each node group runs across multiple Availability Zones that you define.

You can add a managed node group to new or existing clusters using the Amazon EKS console, `eksctl`, AWS CLI; AWS API, or infrastructure as code tools including AWS CloudFormation.
Nodes launched as part of a managed node group are automatically tagged for auto-discovery by the Kubernetes cluster autoscaler.
You can use the node group to apply Kubernetes labels to nodes and update them at any time.

There are no additional costs to use Amazon EKS managed node groups, you only pay for the AWS resources you provision. These include Amazon EC2 instances, Amazon EBS volumes, Amazon EKS cluster hours, and any other AWS infrastructure.
There are no minimum fees and no upfront commitments.

#### Managed node groups concepts

Amazon EKS managed node groups create and manage Amazon EC2 instances for you.

Every managed node is provisioned as part of an Amazon EC2 Auto Scaling group that's managed for you by Amazon EKS.
Moreover, every resource including Amazon EC2 instances and Auto Scaling groups run within your AWS account.

The Auto Scaling group of a managed node group spans every subnet that you specify when you create the group.

Amazon EKS tags managed node group resources so that they are configured to use the Kubernetes Cluster Autoscaler.

You can use a custom launch template for a greater level of flexibility and customization when deploying managed nodes.
For example, you can specify extra kubelet arguments and use a custom AMI.
For more information, see Customizing managed nodes with launch templates.
If you don't use a custom launch template when first creating a managed node group, there is an auto-generated launch template.
Don't manually modify this auto-generated template or errors occur.

#### Managed node group capacity types

When creating a managed node group, you can choose either the On-Demand or Spot capacity type.
Amazon EKS deploys a managed node group with an Amazon EC2 Auto Scaling group that either contains only On-Demand or only Amazon EC2 Spot Instances.
You can schedule Pods for fault tolerant applications to Spot managed node groups, and fault intolerant applications to On-Demand node groups within a single Kubernetes cluster.
By default, a managed node group deploys On-Demand Amazon EC2 instances.

##### On-Demand

With On-Demand Instances, you pay for compute capacity by the second, with no long-term commitments.

## Cluster Authentication

Amazon EKS uses IAM to provide authentication to your Kubernetes cluster (through the `aws eks get-token` command, available in version `1.16.156` or later of the AWS CLI, or the AWS IAM Authenticator for Kubernetes), but it still relies on native Kubernetes Role Based Access Control (RBAC) for authorization.
This means that IAM is only used for authentication of valid IAM entities.
All permissions for interacting with your Amazon EKS cluster's Kubernetes API is managed through the native Kubernetes RBAC system.
The following picture shows this relationship.

### Enabling IAM principal access to your cluster

Access to your cluster using IAM principals is enabled by the AWS IAM Authenticator for Kubernetes, which runs on the Amazon EKS control plane.
The authenticator gets its configuration information from the aws-auth ConfigMap. For all aws-auth ConfigMap settings, see Full Configuration Format on GitHub.

#### Add IAM principals to your Amazon EKS cluster

When you create an Amazon EKS cluster, the IAM principal that creates the cluster is automatically granted `system:masters` permissions in the cluster's role-based access control (RBAC) configuration in the Amazon EKS control plane.
This principal doesn't appear in any visible configuration, so make sure to keep track of which principal originally created the cluster.
To grant additional IAM principals the ability to interact with your cluster, edit the `aws-auth` `ConfigMap` within Kubernetes and create a Kubernetes `rolebinding` or `clusterrolebinding` with the name of a `group` that you specify in the `aws-auth` `ConfigMap`.

##### To add an IAM principal to an Amazon EKS cluster

#### Apply the `aws-auth` `ConfigMap` to your cluster

The `aws-auth` `ConfigMap` is automatically created and applied to your cluster when you create a managed node group or when you create a node group using `eksctl`.
It is initially created to allow nodes to join your cluster, but you also use this `ConfigMap` to add role-based access control (RBAC) access to IAM principals.
If you've launched self-managed nodes and haven't applied the aws-auth ConfigMap to your cluster, you can do so with the following procedure.
