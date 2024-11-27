---
layout: post
title: Pods stuck in the container creating phase due to failure in assigning an IP address to the container
author: Osni Theresa V X
date: 2022-03-09 11:00:00 +0800
categories: [EKS]
tags: [EKS, devops, ]
math: true
mermaid: true
description: Pods stuck in the container creating phase due to failure in assigning an IP address to the container

---
#### Scenario
EKS version 1.20 and with self-managed worker nodes with VPC CNI add-on. Pods are stuck in a ContainerCreating state with the following errors,

```
Event  : sample-deployment   Pod    sample-deployment-76845697dc-vdlzp   sample     Failed to create pod sandbox: rpc error: code = Unknown desc = failed to set up sandbox container "a25a3b78c48165349e770b7e446ff0ca1d39bb61cbf6ba02e1fcf737e23432ee" network for pod "sample-deployment-76845697dc-vdlzp": networkPlugin cni failed to set up pod "sample-deployment-76845697dc-vdlzp_payroll" network: add cmd: failed to assign an IP address to container   FailedCreatePodSandBox
```

#### Solution

Enabled the option ENABLE_PREFIX_DELEGATION to mitigate the issue.

## Prerequisites

* The worker nodes should be Nitro based Instances.
* VPC CNI version must be 1.9.0 or higher.
* VPC subnet must have enough free IP addresses.

1. Confirm the current version VPC CNI plugin.
```
kubectl describe daemonset aws-node -n kube-system | grep Image
```
2. Enable the ENABLE_PREFIX_DELEGATION parameter.
```
kubectl edit daemonset aws-node -n kube-system

```

3. Modify the Environment Variable.
```
containers:
  - name: aws-vpc-cni
    image: 602401143452.dkr.ecr.us-west-2.amazonaws.com/amazon-k8s-cni:v1.11.5
    env:
      - name: ENABLE_PREFIX_DELEGATION
        value: "true"
```
Save and Exit.

4. Verify the env variable.
```
kubectl describe daemonset aws-node -n kube-system | grep ENABLE_PREFIX_DELEGATION

```


This should resolve the issue and pods will be allocated once the nodes are ready. 
