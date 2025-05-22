---
longform:
  format: single
  title: Setup EKS
title: Setup EKS
---
## Setup EKS cluster

```
eksctl create cluster --name kubestack --region us-east-1
```

## Install AWS EBS CSI Driver

```
helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
```

```
helm repo update
```

```
helm upgrade --install aws-ebs-csi-driver --namespace kube-system aws-ebs-csi-driver/aws-ebs-csi-driver
```

```
kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-ebs-csi-driver
```

## Update node role with AmazonEBSCSIDriverPolicy Policy

Here we are going to update the node role with AmazonEBSCSIDriverPolicy policy to allow AWS EC2 service to trigger AWS EBS volume provisioning request.

![[CSI Driver.png]]

## Apply Dynamic provisioning manifests

Now we will try to take sample example to provision the EBS volume dynamically and utilize it as the volume store for the kubernetes pod. 

Let’s clone the repository and apply the manifests for dynamic-provisioning as shown below.

```
git clone https://github.com/kubernetes-sigs/aws-ebs-csi-driver.git 

```

```
cd aws-ebs-csi-driver/examples/kubernetes/dynamic-provisioning
```

```
kubectl apply -f manifests/
```

## Delete EKS Cluster

```
eksctl delete cluster --name kubestack --region us-east-1
```