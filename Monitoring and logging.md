# Monitoring and Logging Documentation

## Overview
This document describes the setup for monitoring the health and status of the system, specifically CPU spikes, storage utilization, and Docker Swarm cluster status. We use AWS CloudWatch to monitor these metrics and leverage AWS SNS (Simple Notification Service) for alerting in case of failures or under-provisioned nodes in the Docker Swarm cluster.

---

## Key Components:
1. **AWS CloudWatch** – Monitors CPU usage, storage utilization, and Docker Swarm cluster health.
2. **Shell Script** – Runs on the master node to gather status data for monitoring.
3. **Docker Swarm Cluster** – Cluster of Docker nodes that are monitored for node health.
4. **AWS SNS** – Sends alerts when conditions are met (e.g., node failures or under-provisioned cluster).

---

## Components in Detail:

### 1. CloudWatch Monitoring Metrics:

#### CPU Spike Monitoring:
- CloudWatch tracks the CPU usage on the master node.
- CPU usage exceeding a set threshold (e.g., 50%) triggers an alert.

#### Storage Utilization:
- CloudWatch monitors disk space on the master node.
- Alerts are sent when disk space exceeds a defined threshold (e.g., 50% used).

---

### 2. Shell Script on Master Node:
A shell script is set up on the master node to collect system metrics and check the status of the Docker Swarm cluster.

- **Docker Swarm Cluster Health**: The script runs `docker node ls` to check the number of active nodes in the Swarm cluster.
- If the number of active nodes drops below 4, an alert is triggered in SNS.

---

### 3. AWS SNS Alerts:
- An SNS topic is created to handle alerts.
- The script triggers an SNS notification when any monitored metric exceeds its threshold (CPU, storage, or node count).

#### Example SNS Notification:
- "CPU usage exceeds 80%: 90%"
- "Storage usage exceeds 85%: 90%"
- "Number of nodes in Docker Swarm is less than 4"

#### Example AWS SNS Setup:

- **Create SNS Topic**:
  - Go to the SNS console and create a topic (e.g., `DockerSwarmAlerts`).
  - Set up subscriptions to the SNS topic (e.g., email or SMS).
  
- **Script Integration**:
  - In the script, replace `<SNS_TOPIC_ARN>` with the actual ARN of the SNS topic.

---

## Workflow:

### 1. Monitoring:
- The shell script is scheduled to run at regular intervals (e.g., every 5 minutes) using cron jobs on the master node.
- The script checks CPU, storage, and Docker Swarm status.

### 2. Alerting:
- If the monitored metrics exceed the thresholds (e.g., CPU > 80%, Storage > 85%, or Docker Swarm nodes < 4), the script triggers an SNS notification.
- The SNS notification sends alerts to the subscribed recipients (e.g., email, SMS).

### 3. Resolution:
- The team receives an alert through SNS, prompting immediate action to resolve the issue (e.g., scaling the Docker Swarm cluster or freeing up storage space).

---

## Conclusion:
This setup ensures that the infrastructure is continuously monitored for potential issues related to CPU, storage, and the health of the Docker Swarm cluster. Alerts are promptly sent through AWS SNS to keep the team informed and enable quick resolution of issues.
