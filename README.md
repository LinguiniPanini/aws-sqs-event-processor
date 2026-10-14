# AWS SQS Event-Driven Order Processing System

[![AWS SQS](https://img.shields.io/badge/Messaging-AWS%20SQS-FF4F8B?logo=amazon-aws&logoColor=white)](https://aws.amazon.com/sqs/)
[![AWS EC2](https://img.shields.io/badge/Compute-Amazon%20EC2-FF9900?logo=amazon-aws&logoColor=white)](https://aws.amazon.com/ec2/)
[![Amazon RDS](https://img.shields.io/badge/Database-Amazon%20RDS%20PostgreSQL-336791?logo=postgresql&logoColor=white)](https://aws.amazon.com/rds/)
[![AWS IAM](https://img.shields.io/badge/Security-Least%20Privilege%20IAM-DD344C?logo=amazon-aws&logoColor=white)](https://aws.amazon.com/iam/)
[![Python](https://img.shields.io/badge/Runtime-Python%203.11-3776AB?logo=python&logoColor=white)](https://www.python.org/)

An asynchronous, event-driven ordering system engineered to absorb massive peak concurrency traffic by decoupling order ingestion from transactional database persistence using **Amazon SQS**, an **Amazon EC2 worker daemon**, **Amazon RDS PostgreSQL**, and **IAM Least-Privilege Access Controls**.

---

## Architecture Diagram

<img width="1373" height="549" alt="Architecture Diagram" src="https://github.com/user-attachments/assets/d8689f8d-1bb6-4ff7-a4ed-f418979d4a40" />

### Component Breakdown

| Component | AWS Service | Implementation Details |
| :--- | :--- | :--- |
| **Ingestion Producer** | AWS CLI / REST Client | Dispatches order payloads (e.g. `Latte\|2026-02-16 13:39:45`) |
| **Message Buffer** | **Amazon SQS** | Standard queue (`coffee-orders-queue`), `VisibilityTimeout=30s`, `Long Polling=20s` |
| **Consumer Daemon** | **Amazon EC2** (`t3.micro`) | `consumer.py` executing continuous long-polling daemon |
| **Persistence Layer** | **Amazon RDS** | PostgreSQL 15 engine, managed `coffee_orders` relation table |
| **Security Governance**| **AWS IAM Role** | Attached Instance Profile scoped strictly to `sqs:ReceiveMessage` and `sqs:DeleteMessage` |

---

## Architectural Mechanics & Failure Handling

1. **Traffic Decoupling:** In peak demand hours, sudden transaction spikes exceed synchronous DB write limits. Ingesting requests directly into Amazon SQS buffers burst traffic with zero request drops.
2. **Efficient Long Polling:** The EC2 worker consumes messages with a 20-second long-polling window (`receive_message`), minimizing empty polling API calls and optimizing AWS operational costs.
3. **Strict Transactional Integrity & Rollback:**
   - Worker receives message from SQS.
   - Worker parses payload and executes SQL `INSERT` inside a database transaction block.
   - Only after a successful `commit` is the message deleted from SQS (`delete_message`).
   - If an error or crash occurs, database `rollback` executes, leaving the message in SQS. After the 30s `VisibilityTimeout` elapses, the message reappears automatically for retry processing.
4. **Least-Privilege Security:** No AWS access keys are stored on the server. The EC2 instance assumes an attached IAM Role scoped exclusively to the specific SQS queue ARN.

---

## Video Walkthrough & Demo

* **Architecture & Live Demo Video:** [Watch on YouTube](https://youtu.be/OgofPu4OWu4)

---

## Local Setup & Deployment

### Run Consumer
```bash
pip install -r requirements.txt
export DB_HOST="your-rds-endpoint.rds.amazonaws.com"
export DB_NAME="coffeedb"
export DB_USER="dbadmin"
export DB_PASS="secret"
export SQS_QUEUE_URL="https://sqs.us-east-1.amazonaws.com/123456789012/coffee-orders-queue"

python consumer.py
```

---

## Author
**Luis Alberto González Escamilla**  
Software & Systems Engineer · AI Agents & Cloud Solutions
