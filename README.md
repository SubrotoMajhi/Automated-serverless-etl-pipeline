# Serverless AWS ETL Pipeline – Project Report

**Interview / Portfolio Project**  
**Author:** Subroto  
**Date of Implementation:** June 18, 2026  
**Region:** us-east-1 (N. Virginia)  
**AWS Account:** 570056717368  

---

## 1. Project Overview

This project implements a **fully serverless, event-driven ETL (Extract-Transform-Load) pipeline** on AWS.  

Raw CSV data (Titanic dataset) is uploaded to an Amazon S3 bucket. An S3 event triggers a Lambda function. An AWS Glue Visual ETL job performs data cleansing (Drop Duplicates). Job state changes and object-created events are routed via Amazon EventBridge to an SNS topic, which delivers real-time email notifications. All activity is observable through Amazon CloudWatch Logs.

### Architecture Summary

```
[User Upload CSV]
       │
       ▼
┌──────────────────┐     S3 Event      ┌─────────────────┐
│  Amazon S3       │ ────────────────► │  AWS Lambda     │
│  server-aws-etl  │                   │  activate-s3    │
│  /extract        │                   └────────┬────────┘
│  /load           │                            │
└────────┬─────────┘                            │
         │                                      │
         │                                      ▼
         │                             ┌─────────────────┐
         │                             │  CloudWatch     │
         │                             │  Logs           │
         │                             └─────────────────┘
         │
         ▼
┌──────────────────┐     Glue Job      ┌─────────────────┐
│  AWS Glue Studio │ ────────────────► │  Transformed    │
│  s3-glue-active  │   (Drop Duplicates)│  data in S3     │
└────────┬─────────┘                   └─────────────────┘
         │
         │ Job State Change
         ▼
┌──────────────────┐     SNS Topic     ┌─────────────────┐
│  EventBridge     │ ────────────────► │  Email (Gmail)  │
│  Rules           │   s3-glue-notifier│  Notifications  │
└──────────────────┘                   └─────────────────┘
```

### Key Technologies

| Service            | Purpose                                      |
|--------------------|----------------------------------------------|
| Amazon S3          | Data lake (raw / extract / load folders)     |
| AWS Lambda         | Event-driven activation on S3 object create  |
| IAM                | Least-privilege roles & policies             |
| AWS Glue Studio    | Visual ETL job with Drop Duplicates          |
| Amazon EventBridge | Event routing (S3 + Glue state changes)      |
| Amazon SNS         | Email notification delivery                  |
| Amazon CloudWatch  | Logging & monitoring                         |

---

## 2. Screenshots – Sequential Walkthrough (Start → End)

All screenshots are in **PNG** format.  
Original sequence numbers are preserved so the narrative maps directly to the files in `Screenshots/`.

### Phase 1 – Amazon S3 Setup (Screenshots 51–69)

| Range | What happens |
|-------|--------------|
| **51–52** | Landing on the Amazon S3 console “Get started” page. User is in the Free Tier account (`subroto`). |
| **53–56** | Create bucket wizard opens. Bucket name entered: **`server-aws-etl`**. General purpose bucket type selected, Global namespace, Region `us-east-1`. |
| **57–59** | Object Ownership, Block Public Access, Bucket Versioning, and Encryption settings configured (defaults kept for security). |
| **60** | Create folder dialog – folder name **`load`** is being created inside the new bucket. |
| **62–64** | Folder creation and bucket overview after successful creation of `server-aws-etl`. |
| **65–69** | Additional folder structure preparation and navigation back to the bucket. |

**Outcome:** Clean S3 data lake with organized prefixes ready for raw data ingestion.

---

### Phase 2 – AWS Lambda Function Creation (Screenshots 70–100)

| Range | What happens |
|-------|--------------|
| **70–72** | “Create function” page – Author from scratch selected. Function name will become **`activate-s3`**. Runtime initially shown as Node.js, later switched to **Python 3.14**. |
| **73–79** | IAM execution role creation. Custom role **`activate-s3-role-u5zvmyff`** is created and attached. |
| **80–85** | Lambda code editor. Simple Python handler that prints the incoming event and returns a 200 response (“Hello from Lambda!”). |
| **86–92** | Deploy & Test. Test event named `s3-put` is executed successfully. |
| **93–100** | Configuration of S3 trigger (Object Created events on the `server-aws-etl` bucket) and verification that the function is now event-driven. |

**Outcome:** Lambda function is live, has the correct execution role, and is ready to react to S3 uploads.

---

### Phase 3 – IAM Role & Permissions Hardening (Screenshots 101–115)

| Range | What happens |
|-------|--------------|
| **101–108** | Detailed view of the Lambda execution role in IAM. Policies successfully attached. |
| **109–115** | Trust relationships and additional permissions reviewed to ensure the role can write to CloudWatch Logs and interact with S3. |

**Outcome:** Least-privilege IAM role is correctly configured.

---

### Phase 4 – CloudWatch Log Groups (Screenshots 116–130)

| Range | What happens |
|-------|--------------|
| **116–122** | CloudWatch Log group `/aws/lambda/activate-s3` is automatically created after the first invocation. |
| **123–130** | Log events inspected; Lambda execution logs confirm successful triggers. |

**Outcome:** Full observability of Lambda invocations is in place.

---

### Phase 5 – AWS Glue Visual ETL Job (Screenshots 131–185)

| Range | What happens |
|-------|--------------|
| **131–140** | AWS Glue Studio opened. New Visual ETL job created. |
| **141–155** | Data Source node configured → S3 bucket **`server-aws-etl`**. |
| **156–165** | Transform node **Drop Duplicates** added to the job graph. |
| **166–175** | Data Target node configured → write cleaned data back to S3 (typically under a `load/` or processed prefix). |
| **176–185** | Job named **`s3-glue-active`**, saved, and first successful run completed (status: Succeeded, ~1–2 min runtime, 10 DPUs, G.1X workers). |

**Outcome:** Visual ETL pipeline that removes duplicate rows from the Titanic CSV is production-ready.

---

### Phase 6 – EventBridge + SNS Notification Setup (Screenshots 186–230)

| Range | What happens |
|-------|--------------|
| **186–200** | Amazon EventBridge rule creation. Rule matches S3 “Object Created” and Glue “Job State Change” events. |
| **201–215** | Target configured as SNS topic **`s3-glue-notifier`**. New execution role for EventBridge → SNS is created. |
| **216–230** | SNS topic subscription confirmed (email endpoint). Protocol = Email, endpoint = user’s Gmail. |

**Outcome:** Real-time email notifications are delivered whenever an object is created in S3 or a Glue job finishes.

---

### Phase 7 – End-to-End Test & Verification (Screenshots 231–320)

| Range | What happens |
|-------|--------------|
| **231–250** | CloudWatch Logs show S3 event details (bucket `s3-event-generate-serverless`, object `titanic11.csv`). |
| **251–270** | Glue job runs tab – both historical runs show **Succeeded**. |
| **271–290** | Final data upload verification: `titanic.csv` successfully uploaded to `s3://server-aws-etl/extract/`. |
| **291–310** | Additional monitoring of EventBridge and SNS delivery. |
| **311–320** | **Gmail inbox** receives AWS Notification Messages:  
  - “Object Created” event  
  - “Glue Job State Change – SUCCEEDED”  

**Outcome:** Complete closed-loop demonstration – upload → Lambda → Glue ETL → Email notification.

---

## 3. Project Highlights for Interview

- **Serverless-first design** – zero EC2/EMR management.
- **Event-driven architecture** – S3 → Lambda + EventBridge → SNS.
- **Visual ETL** – no handwritten Spark code; Glue Studio graph is easy to explain and maintain.
- **Security** – IAM roles with least privilege, Block Public Access enabled on S3.
- **Observability** – CloudWatch Logs + email notifications give both technical and business visibility.
- **Cost-efficient** – Free Tier friendly (S3, Lambda, Glue, SNS, EventBridge all have free tiers).

---

## 4. How to Reproduce (High-Level)

1. Create S3 bucket `server-aws-etl` with folders `extract/` and `load/`.
2. Create Lambda `activate-s3` (Python) with S3 trigger on the bucket.
3. Create Glue Visual ETL job `s3-glue-active` (Source S3 → Drop Duplicates → Target S3).
4. Create SNS topic `s3-glue-notifier` and subscribe your email.
5. Create EventBridge rules that forward S3 Object Created and Glue Job State Change events to the SNS topic.
6. Upload a CSV (e.g. Titanic) to `s3://server-aws-etl/extract/` and watch the pipeline + email notification fire.

---

## 5. Future Improvements

- Add Glue Data Quality rules.
- Parameterize the Glue job (job bookmarks, schema evolution).
- Use Step Functions for multi-stage orchestration.
- Add Athena table for ad-hoc querying of the cleaned data.
- Infrastructure as Code (Terraform / AWS CDK) for full reproducibility.

---


