# Almanax Auditor

## Overview
Almanax Auditor is a web-based platform that allows users to upload their Solidity smart contracts for auditing. The auditing is performed by a black box LLM service that analyzes the contract for vulnerabilities and best practices. The results are then presented to the users in an understandable format.

## Architecture Diagram

```plaintext
+-----------------------+
|    User Interface     |
|  (Web Application)    |
+-----------------------+
           |
           v
+-----------------------+
|   API Gateway         | <--- User authentication (AWS Cognito)
|   (AWS API Gateway)   |
+-----------------------+
           |
           v
+-----------------------+
|  Audit Request Queue  | <--- SQS for job management
|   (AWS SQS)          |
+-----------------------+
           |
           v
+-----------------------+
|  Audit Service        | <--- LLM service (black box)
| (AWS Lambda / EC2)    |
+-----------------------+
           |
           v
+-----------------------+
|   Storage System      | <--- S3 for smart contracts and reports
|  (AWS S3)            |
+-----------------------+
           |
           v
+-----------------------+
|  Database             | <--- RDS for metadata storage
| (AWS RDS)            |
+-----------------------+
           |
           v
+-----------------------+
|  CDN                  | <--- Fast content delivery for reports
| (AWS CloudFront)     |
+-----------------------+
```

## Key Components
1. User Interface (Web Application):

* Functionality: Allows users to upload contracts, view reports, and manage their submissions.
* Security: Implements user authentication via AWS Cognito.

2. API Gateway:

* Role: Acts as a single entry point for all requests.
* Security: Handles user authentication, input validation, and rate limiting.

3. Audit Request Queue:

* Implementation: Uses AWS SQS to queue audit requests.
* Benefits: Ensures reliable message delivery and decouples the front-end from the auditing process.

4. Audit Service:

* Description: This is the black box LLM service that processes the smart contracts.
* Scalability: Deployed on AWS Lambda or EC2 to allow for scaling based on demand.

5. Storage System:

* AWS S3: Used for storing uploaded smart contracts and generated audit reports.
* Versioning: Implements version control for contracts and audit results.

6. Database:

* AWS RDS: Stores metadata about users, contracts, and audit reports.
* Backup and Recovery: Automated backups to ensure data integrity.

7. Content Delivery Network (CDN):

* AWS CloudFront: Provides fast and secure access to audit reports.
* Caching: Reduces latency for frequently accessed reports.

## Workflow
1. User Uploads Contract:

* User uploads a Solidity smart contract via the UI.
* The UI sends the contract to the API Gateway.

2. Queue Audit Request:

The API Gateway forwards the request to the SQS.
The contract is stored in S3 for future reference.

3. Audit Process:

* A Lambda function (or EC2 instance) triggers on new SQS messages.
* The contract is sent to the black box LLM service for auditing.

4. Report Generation:

* The LLM service analyzes the contract for vulnerabilities (e.g., reentrancy, integer overflow).
* Audit results are generated and stored in S3.

5. Notification:

* Upon completion, a notification (e.g., via email) is sent to the user.
* The report is accessible via the CDN for quick retrieval.

6. Review and Feedback:
* Users can review the report and provide feedback, which can be stored in RDS for improving future audits.

## Security Considerations
* Input Validation: Ensure all inputs are validated at the API level to prevent injection attacks.
* Data Encryption: Use encryption at rest and in transit for sensitive data.
* Access Control: Implement strict IAM roles for AWS resources to limit access based on user roles.
* Monitoring and Logging: Use AWS CloudWatch for monitoring and logging all API requests and audit processes.

## Addressing Common Vulnerabilities
Utilizing the Blockchain Common Vulnerability List, the LLM service should specifically check for:

* Reentrancy Attacks: Ensure functions that modify state are protected.
* Integer Overflow/Underflow: Use SafeMath libraries.
* Access Control Issues: Verify function visibility and modifiers.
* Gas Limit and Loops: Analyze for potential excessive gas consumption.

## Handling Large Scale Audits
1. Data Storage:
* Use Amazon S3 for scalable object storage.
* Implement S3 Intelligent-Tiering to automatically optimize costs by moving data between two access tiers when access patterns change.
* Use S3 Glacier for archiving older audit reports to reduce storage costs.

2. Data Processing:
* Use AWS Lambda for serverless computing to process audit requests dynamically.
* Consider using AWS Batch for batch processing of contracts to handle large volumes efficiently.

3. Database Management:
* Use Amazon RDS or Amazon DynamoDB for managing metadata.
* Implement sharding or partitioning strategies to handle high throughput and large data sizes.

4. Data Transfer and CDN:
* Utilize Amazon CloudFront as a CDN to distribute reports efficiently, minimizing latency for global users.
* Use AWS Direct Connect for high-speed data transfer if audits involve large datasets.

5. Scaling Strategy:
* Implement autoscaling groups for EC2 instances handling the LLM service.
* Use Amazon Elastic Kubernetes Service (EKS) for orchestrating containers that can scale dynamically based on load.

## Reasonable Limitations
* Data Ingestion Rate: Assess the expected number of audits per second. For example, if you plan to handle 10,000 audits per second and each audit generates an average of 10MB of data, that results in 100GB of data per second.

* Storage Requirements: Determine data retention policies. For example, retaining data for five years would mean 100GB/sec translates to 15.7PB/year.

* Cost Management: Implement budgets and alerts in AWS Budgets to manage spending.

## AWS Pricing Calculation
### Example Costs for Services
1. Amazon S3:

* Standard Storage: $0.023 per GB.
* Intelligent-Tiering: $0.0125 per GB for infrequent access.
* Glacier: $0.004 per GB.

2. AWS Lambda:
* $0.20 per 1 million requests + $0.00001667 for every GB-second used.

3. Amazon RDS (MySQL):
* db.t3.medium instance: ~$0.0416 per hour + storage costs.

4. Amazon CloudFront:
* $0.085 per GB for the first 10TB/month.

5. AWS Data Transfer:
* The first 1GB out is free, then $0.09/GB for up to 10TB.

## Cost Estimation
### Assumptions:
* Data Storage: 10TB in S3 Standard.
* Lambda Requests: 10 million requests.
* RDS Usage: 1 db.t3.medium instance for 720 hours.
* CloudFront: 5TB data transfer.

### Weekly Costs
1. S3 Storage:
* 10TB * $0.023/GB = $230 per week.

2. Lambda:
* 10 million requests: $0.20 = $0.20 per week.

3. RDS:
* $0.0416/hour * 720 hours = $29.95 per week.

4. CloudFront:
* 5TB * $0.085/GB = $425 per week.

5. Total Weekly Cost:
* $230 (S3) + $0.20 (Lambda) + $29.95 (RDS) + $425 (CloudFront) = $685.15

### Monthly Costs
* Total Monthly Cost = Weekly cost * 4 = $2,740.60

### Daily Costs
* Total Daily Cost = Weekly cost / 7 = $97.87

### Summary
| Timeframe  | Cost |
| ------------- | ------------- |
| Daily  |$97.87  |
| Weekly  | $685.15  |
| Monthly  | $2,740.60  |

## Final Remarks
The Almanax Auditor platform provides a robust, scalable, and secure solution for smart contract auditing. Using AWS infrastructure and best practices, along with a specialized LLM service, it aims to enhance the reliability of smart contracts in the blockchain ecosystem. Managing several petabytes of data for large-scale audits involves strategic planning in storage, processing, and cost management. The estimated pricing provides a baseline for operational budgeting, considering that actual costs may vary based on usage patterns and data transfer rates.
