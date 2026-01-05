# PROPOSAL-kafka-on-eks-with-strimzi

## 1. Why Strimzi on EKS
Strimzi is an open‑source Kubernetes operator that manages Apache Kafka clusters using Kubernetes-native CRDs, automating provisioning, configuration changes, rolling upgrades, and scaling.
On EKS, this provides a repeatable, declarative way to run Kafka 4.x without commercial licenses, while aligning with our existing Kubernetes/IaC practices.

## Advantages of Strimzi
  * Kubernetes-native lifecycle
      Manage Kafka clusters, topics, users, and Connect via CRDs, which fit naturally into GitOps (ArgoCD/Flux) workflows.
      Automated rolling upgrades, health checks, and broker replacement reduce manual operational work.
 ​

  * Operational safety and consistency
      Built-in patterns for HA (multi-broker, multi-AZ), PodDisruptionBudgets, and controlled node draining using Strimzi drain-cleaner and Cruise Control for rebalancing.
​      Strong default security: TLS for inter-broker and client traffic, SCRAM/OAuth/mTLS support, and NetworkPolicies.
​
  * EKS alignment
      Works with standard EKS primitives (nodegroups, EBS StorageClasses, IAM roles), and is referenced by AWS “Kafka on EKS” blueprints.
​      Fits with existing monitoring stack (Prometheus/Grafana) via Kafka and operator metrics.
​
