---
layout: post
title: Bash script to create a cluster with multisubnet in EKS
author: Osni Theresa V X
date: 2024-07-14 11:00:00 +0800
categories: [EKS]
tags: [EKS, devops, ]
math: true
mermaid: true
description: Bash script to create a cluster with multisubnet in EKS

---
#### Prerequisites
* eksctl installed

* awscli configured with proper IAM permissions

* Subnets should already exist or be created via the config file

#### Script

```
#!/bin/bash

read -p "Enter EKS cluster name: " CLUSTER_NAME
read -p "Enter AWS region (e.g., us-west-2): " REGION
read -p "Kubernetes version (e.g., 1.29): " VERSION

echo "Select subnet configuration:"
select NET_TYPE in "private" "public" "mixed"; do
  case $NET_TYPE in
    private|public|mixed) break ;;
    *) echo "Invalid option. Choose 1, 2, or 3." ;;
  esac
done

read -p "Use existing VPC? (y/n): " USE_EXISTING

if [[ $USE_EXISTING == "y" ]]; then
  read -p "Enter VPC ID: " VPC_ID
  read -p "Enter comma-separated subnet IDs (e.g., subnet-abc,subnet-def): " SUBNET_IDS
fi

# Generate eksctl config
cat <<EOF > eksctl-config.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: $CLUSTER_NAME
  region: $REGION
  version: "$VERSION"
EOF

if [[ $USE_EXISTING == "y" ]]; then
  echo "vpc:" >> eksctl-config.yaml
  echo "  id: $VPC_ID" >> eksctl-config.yaml
  echo "  subnets:" >> eksctl-config.yaml
  echo "    $NET_TYPE:" >> eksctl-config.yaml
  for id in $(echo $SUBNET_IDS | tr "," "\n"); do
    echo "      $(aws ec2 describe-subnets --subnet-ids $id --region $REGION \
      --query 'Subnets[0].AvailabilityZone' --output text): { id: $id }" >> eksctl-config.yaml
  done
else
  cat <<EOF >> eksctl-config.yaml
vpc:
  cidr: "10.0.0.0/16"
  subnets:
    $NET_TYPE:
      ${REGION}a: { cidr: "10.0.1.0/24" }
      ${REGION}b: { cidr: "10.0.2.0/24" }
      ${REGION}c: { cidr: "10.0.3.0/24" }
  nat:
    gateway: Single
EOF
fi

cat <<EOF >> eksctl-config.yaml

managedNodeGroups:
  - name: ${NET_TYPE}-ng
    instanceType: t3.medium
    desiredCapacity: 2
    privateNetworking: true
EOF

# Show final config
echo -e "\nGenerated eksctl config:\n"
cat eksctl-config.yaml

# Confirm and run
read -p "Do you want to create the cluster now? (y/n): " CONFIRM
if [[ $CONFIRM == "y" ]]; then
  eksctl create cluster -f eksctl-config.yaml
else
  echo "Cluster creation cancelled."
fi
```

Thats it. Your eks cluster is ready.
