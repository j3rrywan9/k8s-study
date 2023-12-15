# EKS Upgrade Notes

## Deprecated API Migration Guide

https://kubernetes.io/docs/reference/using-api/deprecation-guide/

## v1.25

### Amazon VPC CNI plugin for Kubernetes

The Amazon VPC CNI plugin for Kubernetes add-on is deployed on each Amazon EC2 node in your Amazon EKS cluster.
The add-on creates elastic network interfaces and attaches them to your Amazon EC2 nodes.
The add-on also assigns a private IPv4 or IPv6 address from your VPC to each Pod and service.

We recommend adding the Amazon EKS type of the add-on to your cluster instead of using the self-managed type of the add-on.

#### How to remove the CRD created by the outdated `aws-vpc-cni` Helm chart

A CRD named `eniconfigs.crd.k8s.amazonaws.com` with deprecated API version `apiextensions.k8s.io/v1beta1` was identified by Pluto, whose `metadata` has the following:
```yaml
annotations:
  kubectl.kubernetes.io/last-applied-configuration: |
    {"apiVersion":"apiextensions.k8s.io/v1beta1","kind":"CustomResourceDefinition","metadata":{"annotations":{},"labels":{"app.kubernetes.io/instance":"aws-vpc-cni-main-production-red-usw2","app.kubernetes.io/managed-by":"Helm","app.kubernetes.io/name":"aws-node","app.kubernetes.io/version":"v1.8.0","helm.sh/chart":"aws-vpc-cni-1.1.7","k8s-app":"aws-node"},"name":"eniconfigs.crd.k8s.amazonaws.com"},"spec":{"group":"crd.k8s.amazonaws.com","names":{"kind":"ENIConfig","plural":"eniconfigs","singular":"eniconfig"},"scope":"Cluster","versions":[{"name":"v1alpha1","served":true,"storage":true}]}}
creationTimestamp: "2021-07-01T18:44:02Z"
```
Apparently, the CRD was created using the outdated `aws-vpc-cni` Helm chart back to 2021.

The CRD provides the following CR:
```zsh
k api-resources | grep eniconfigs
eniconfigs                                            crd.k8s.amazonaws.com/v1alpha1         false        ENIConfig
```

Delete the CRD:
```zsh
k get crds | grep eniconfigs.crd.k8s.amazonaws.com

k get eniconfigs.crd.k8s.amazonaws.com

k delete crd eniconfigs.crd.k8s.amazonaws.com
```

Now the following command should return nothing as the CRD is deleted:
```zsh
k api-resources | grep eniconfigs
```

At this point, Pluto should no longer report this CRD.

### [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)

> We recommend using Cluster Autoscaler with the Kubernetes control plane (previously referred to as master) version for which it was meant.

Based on the above statement, I'm supposed to upgrade to version `v1.25.3`, which can be found [here](https://github.com/kubernetes/autoscaler/releases/tag/cluster-autoscaler-1.25.3).
However, there is no Helm chart version shipped with this particular CA release:
```zsh
helm repo add cluster-autoscaler https://kubernetes.github.io/autoscaler

helm search repo cluster-autoscaler --versions

NAME                                       	CHART VERSION	APP VERSION	DESCRIPTION
cluster-autoscaler/cluster-autoscaler      	9.34.0       	1.28.2     	Scales Kubernetes worker nodes within autoscali...
cluster-autoscaler/cluster-autoscaler      	9.33.0       	1.27.2     	Scales Kubernetes worker nodes within autoscali...
cluster-autoscaler/cluster-autoscaler      	9.32.1       	1.27.2     	Scales Kubernetes worker nodes within autoscali...
cluster-autoscaler/cluster-autoscaler      	9.32.0       	1.27.2     	Scales Kubernetes worker nodes within autoscali...
cluster-autoscaler/cluster-autoscaler      	9.31.0       	1.27.2     	Scales Kubernetes worker nodes within autoscali...
cluster-autoscaler/cluster-autoscaler      	9.30.1       	1.27.2     	Scales Kubernetes worker nodes within autoscali...
cluster-autoscaler/cluster-autoscaler      	9.30.0       	1.27.2     	Scales Kubernetes worker nodes within autoscali...
cluster-autoscaler/cluster-autoscaler      	9.29.5       	1.27.2     	Scales Kubernetes worker nodes within autoscali...
cluster-autoscaler/cluster-autoscaler      	9.29.4       	1.27.2     	Scales Kubernetes worker nodes within autoscali...
cluster-autoscaler/cluster-autoscaler      	9.29.3       	1.27.2     	Scales Kubernetes worker nodes within autoscali...
cluster-autoscaler/cluster-autoscaler      	9.29.2       	1.27.2     	Scales Kubernetes worker nodes within autoscali...
cluster-autoscaler/cluster-autoscaler      	9.29.1       	1.27.2     	Scales Kubernetes worker nodes within autoscali...
cluster-autoscaler/cluster-autoscaler      	9.29.0       	1.27.1     	Scales Kubernetes worker nodes within autoscali...
cluster-autoscaler/cluster-autoscaler      	9.28.0       	1.26.2     	Scales Kubernetes worker nodes within autoscali...
cluster-autoscaler/cluster-autoscaler      	9.27.0       	1.24.0     	Scales Kubernetes worker nodes within autoscali...
...
```
Based on the comments found in [this GitHub Issue](https://github.com/kubernetes/autoscaler/issues/5761), I ended up using Helm chart version `9.29.5` with customized CA image tag i.e. `v1.25.3` in the `values.yaml` file.
