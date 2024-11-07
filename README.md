# Infrastructure Architecture Overview

This repository documents the infrastructure architecture deployed in a single AWS region, designed to be replicated across multiple regions for enhanced availability and resilience. The architecture is optimized for scalability, security, and comprehensive monitoring, making it suitable for production-grade applications.

## Architecture Design

### 1. Regional Deployment
This infrastructure is deployed in one AWS region, with an easily replicable design to support **multi-region availability** and **cross-region resilience** if needed. Each component is designed to scale and operate independently, ensuring both fault tolerance and high performance.

### 2. Network Segmentation
   - **Private Subnet**: Hosts the majority of application components, securing critical resources from direct internet exposure and providing an isolated environment for sensitive workloads.
   - **Public Subnet**: Contains only the **Teleport Proxy Cluster**, enabling secure access for internal users to the EKS clusters. This configuration provides multi-user access with audit logging capabilities, which improve over standard bastion hosts that may slow down under heavy load. AWS SSM could also be an alternative, but Teleport’s logging features add an extra security layer by tracking user actions.

### 3. Routing and Internet Access
   - **NAT Gateway**: Connects the private subnet to the public subnet, allowing resources in the private subnet to access the internet securely through the NAT gateway.
   - **Internet Gateway with Squid Proxy**: The public subnet nodes use an Internet Gateway with a **Squid Proxy** in between, enabling fine-grained traffic control, caching, and improved outbound request performance. Squid Proxy restricts access and helps with network security.

### 4. Application Security
   - **AWS WAF**: Positioned in front of two Application Load Balancers (ALBs), AWS WAF filters and blocks malicious requests before they reach application layers. Logs from WAF are stored in S3 for auditing and analysis.
   - **Application Load Balancers**: Two ALBs are deployed, one for the public subnet and another for the private subnet. They work in tandem with the WAF to distribute incoming traffic securely and efficiently.
   - **Routing with NGINX Ingress and Route53**: Requests are routed via **Route53** (domain resolution) through WAF to the ALBs, then forwarded to the NGINX Ingress, which serves as a reverse proxy for Kubernetes services within the EKS clusters.

### 5. Data Storage and Caching
   - **DynamoDB**: Provides high scalability and resilience for application data, supporting partitioned storage and replication with an emphasis on write performance through Eventual Consistency.
   - **Redis Caching**: Before accessing DynamoDB, Redis is checked for requested data, reducing read latency and improving performance by serving frequent requests from cache.
   - **S3 Storage**: S3 is used for storing static content (e.g., images) for application clusters, while logs from the ELK cluster are stored separately for easy access and long-term storage.

### 6. Persistent Storage for Critical Services
   - **EBS for Persistent Storage**: Both the Teleport server cluster and the Prometheus cluster (including Grafana) use EBS volumes for persistent data storage. Additionally, **HashiCorp Vault** leverages EBS for secure storage of its applications.
   - **Encryption**: All nodes and EBS volumes are encrypted using AWS Key Management Service (KMS), enhancing data protection and meeting compliance requirements.

### 7. Scalability and Auto-Scaling
   - **Auto-Scaling**: Auto-scaling is configured for the Teleport proxy cluster and the primary application cluster, enabling them to handle fluctuating user access and ensure responsiveness.
   - **Network CNI**: **Cilium** is selected as the network CNI for its advanced networking capabilities, including support for network policies, layer 7 visibility, and enhanced security controls, which provide granular control over network traffic.

## Monitoring, Logging, and Threat Detection

To ensure complete observability and security, this architecture includes a multi-layered monitoring and logging approach:

1. **CloudTrail, CloudWatch, and GuardDuty**:
   - **CloudTrail** tracks all API activity across AWS, providing detailed logs for compliance and audit purposes.
   - **CloudWatch** monitors system performance metrics, enabling real-time alerts and tracking of critical application and infrastructure metrics.
   - **GuardDuty** continuously scans for potential threats and anomalies, such as unauthorized access or unusual traffic, providing an additional layer of security.

2. **ELK Stack for Log Aggregation**:
   - The architecture includes an **ELK (Elasticsearch, Logstash, and Kibana) stack** to aggregate and analyze application logs in a centralized manner. ELK enables detailed log analysis and visualization, helping to detect issues in real-time. This cluster can be self-managed or managed, depending on requirements.

3. **Prometheus and Grafana for Observability**:
   - A **Prometheus and Grafana cluster** is deployed for real-time observability across all EKS clusters. Prometheus collects metrics from services and applications, while Grafana visualizes these metrics in dashboards for ongoing monitoring of performance, trends, and bottlenecks.

4. **Centralized Log Storage and Analysis**:
   - All logs from CloudTrail, CloudWatch, GuardDuty, ELK, and Prometheus can be sent to **AWS Athena** (AWS’s managed data lake service, similar to BigQuery) for centralized storage and analysis. This enables efficient, long-term querying of logs across all sources, allowing correlation of events and insights across clusters without maintaining separate infrastructure.

## Summary

This architecture is built with scalability, resilience, and security in mind. By combining public and private subnets, AWS managed services (WAF, Route53, DynamoDB), advanced networking (Cilium), and monitoring tools (CloudTrail, CloudWatch, GuardDuty, ELK, Prometheus), the setup effectively balances performance, user experience, and security. This infrastructure is flexible and robust, designed for easy expansion across AWS regions for enhanced availability.

---

Please refer to the respective directories and configuration files in this repository for detailed deployment scripts, configuration, and usage instructions.
