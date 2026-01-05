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
   

 ### 2.2 Test Tools
     * Kafka built-in tools: kafka-producer-perf-test.sh and kafka-consumer-perf-test.sh, packaged in a test “client pod” image.
​     * Load tools like k6 with xk6-kafka, JMeter, or Gatling for more realistic scenarios and scripting.
​     * Observability stack: Prometheus + Grafana dashboards for Kafka and Strimzi operator metrics during tests

 ### 2.3 Step-by-step Test Procedure
     
    * Baseline cluster setup
         * Deploy Kafka via Strimzi with production-like configuration (broker count, partitions, replication factor, disk type/size).
​
    * Define test scenarios
         * Use realistic message sizes, partition counts, key distribution, and compression settings that match our workloads.
​
    * Run incremental load tests
         * Start with low load and gradually ramp up QPS/throughput, recording throughput, latency, and resource metrics.
​         * Push until saturation (e.g., high disk utilization or latency spikes) to determine safe operating headroom.​

    * Stress and failure tests
         * Simulate broker/node failures (drain a node, kill a broker pod) and observe recovery times and impact on latency.
​         * Test partition rebalancing, rolling upgrade simulations, and network disturbances where possible.
​
    * Analysis and tuning loop
         * Adjust key Kafka configs (I/O threads, network threads, batch sizes, linger.ms, log.segment.bytes) and rerun tests.​
         * Document recommended instance types, broker count, max partitions per broker, and expected SLA ranges from these results

## 3. Important Kafka & Strimzi Configuration on EKS

### 3.1 Cluster and Nodegroup Design
   * Dedicated Kafka nodegroup with:
       Instance families optimized for network and disk (e.g., m6i/m7i or r6i), spread across 3 AZs.
       Taints and tolerations so only Kafka workloads schedule on these nodes.
​
   * Storage:
       Use EBS gp3 or io2 with tuned IOPS/throughput (not default gp2).
​       One PVC per broker; size determined from retention, throughput, and headroom targets.
### 3.2 Strimzi Kafka CR Key Settings

  * Broker-level settings (in Kafka.spec.kafka.config):
      * num.network.threads, num.io.threads, log.retention.hours, log.segment.bytes, auto.create.topics.enable, min.insync.replicas.
​      * Listener configuration (internal vs external listeners, TLS, authentication).
​
  * Storage and durability:
    * replication.factor per topic and min.insync.replicas to tolerate broker loss.
​    * PodDisruptionBudgets and max unavailable broker settings from Strimzi.
​
  * Strimzi features:
    * Enable Cruise Control for balancing and node draining integration.
​    * Configure KafkaTopic and KafkaUser CRs to standardize topic defaults and security.

### 3.3 Observability and SLOs
   * Enable Kafka and Strimzi metrics endpoints, scrape via Prometheus, and create SLOs on availability, latency, and message loss.
​   * Capture and alert on lag, offline partitions, ISR changes, and disk usage growth.

## 4. KRaft: How It Works and Why Use It
   * KRaft (Kafka Raft) replaces ZooKeeper with an internal metadata quorum, simplifying the architecture and improving consistency. Kafka 4.x uses KRaft as the default for new clusters, and Strimzi supports KRaft-based deployments.
​​
