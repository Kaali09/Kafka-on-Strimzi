# PROPOSAL-kafka-on-eks-with-strimzi

## 1. Why Strimzi on EKS
Strimzi is an open‑source Kubernetes operator that manages Apache Kafka clusters using Kubernetes-native CRDs, automating provisioning, configuration changes, rolling upgrades, and scaling.
On EKS, this provides a repeatable, declarative way to run Kafka 4.x without commercial licenses, while aligning with our existing Kubernetes/IaC practices.

### Advantages of Strimzi
  * Kubernetes-native lifecycle
      Manage Kafka clusters, topics, users, and Connect via CRDs, which fit naturally into GitOps (ArgoCD/Flux) workflows.
      Automated rolling upgrades, health checks, and broker replacement reduce manual operational work.​

  * Operational safety and consistency
      Built-in patterns for HA (multi-broker, multi-AZ), PodDisruptionBudgets, and controlled node draining using Strimzi drain-cleaner and Cruise Control for rebalancing.
​      Strong default security: TLS for inter-broker and client traffic, SCRAM/OAuth/mTLS support, and NetworkPolicies.
​
  * EKS alignment
      Works with standard EKS primitives (nodegroups, EBS StorageClasses, IAM roles), and is referenced by AWS “Kafka on EKS” blueprints.
​      Fits with existing monitoring stack (Prometheus/Grafana) via Kafka and operator metrics.
​
### Disadvantages / Trade-offs
   * Operator dependency and version coupling
       Kafka versions are tied to Strimzi releases; upgrading Kafka usually means upgrading Strimzi first, then the Kafka cluster CR.
​       Extra component to operate and monitor (operator pods, CRDs, image updates).
​
   * Feature scope vs. commercial distros
       Does not include paid features from Confluent/Instaclustr (UI, turnkey connectors, governance features), which may matter for some teams.
​       Tiered storage and advanced cost-optimization patterns require careful self-implementation and configuration.
​
   * Kubernetes complexity
       Kafka on Kubernetes has storage and noisy-neighbor concerns; it requires careful nodegroup, disk, and network design.

## 2. Performance Testing Plan for Kafka on EKS

### 2.1 Objectives and Metrics
  * Define clear test goals:

    * Target throughput (messages/sec, MB/sec) for key workloads.
​    * Acceptable end-to-end latency (p50, p95, p99).
​    * Resource utilization bounds (CPU, memory, disk I/O, network).
​
  * Core metrics to track:

    * Broker metrics: request-rate, request-latency, bytes-in/out, disk I/O, queue sizes, ISR count.
    * Client metrics: producer/consumer throughput, error rates, retries, batch sizes.
