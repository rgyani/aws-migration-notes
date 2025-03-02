# My notes on AWS Migration services from On-Premise to AWS cloud

## Steps
| Phase | Tools and Services |
|---|---|
| Discovery & Assessment | AWS Application Discovery Service, AWS Trusted Advisor |
| Planning | AWS Well-Architected Framework, AWS Security Hub |
| AWS Setup |	VPC, IAM, AWS Control Tower, Route 53 |
| Migration Execution | AWS MGN, DMS, DataSync, Snowball |
| Testing & Optimization| AWS Compute Optimizer, Auto Scaling, CloudWatch |
| Cutover & Post-Migration |	Route 53, Cost Explorer, CloudFormation |

## Services

### 1. AWS Application Discovery Service (ADS)
AWS Application Discovery Service (ADS) is a tool that helps enterprises discover, analyze, and plan the migration of on-premises infrastructure to AWS. It provides insights into server configurations, utilization metrics, and application dependencies, allowing for a smooth cloud migration.

**Key Features of AWS ADS**
* Automatic Data Collection
    * Uses lightweight agents or agentless connectors **on-premise** to collect:
        * CPU, memory, disk utilization
        * Installed applications
        * Running processes
        * Network dependencies (which servers talk to each other)
* Dependency Mapping
    * Helps you visualize server dependencies to ensure smooth migration.
    * Prevents broken connections between servers, databases, or applications.
* Performance Metrics
    * Provides real-time usage stats (CPU, RAM, Disk I/O, Network traffic).
    * Helps right-size AWS instances to avoid overspending.
* Works with AWS Migration Hub to plan & track migrations.
* Helps decide which AWS services to use (EC2, RDS, Lambda, etc.).

**How AWS ADS Works**
* Step 1: Deploy the Discovery Tool
    * **Agent-Based Discovery**: Installed on individual VMs (Windows/Linux) to collect detailed data.
    * **Agentless Discovery**: Uses VMware vCenter Connector to analyze entire environments without installing agents.
* Step 2: Data Collection & Analysis
    * AWS ADS collects utilization and dependency data over days or weeks.
    * The data is stored in AWS Migration Hub for analysis.
* Step 3: Generate Reports including:
    * Application dependency maps
    * Resource utilization trends
    * Server-to-server communication
* Step 4: Migration Planning
    * Use the data to choose the right AWS instance types and services.
    * Helps decide between Lift-and-Shift, Replatforming, or Refactoring.

**When to Use AWS ADS?**
* Large-scale enterprise migrations
* Understanding application dependencies before migration
* Optimizing AWS costs based on actual server usage
* Reducing downtime risks


### 2. AWS Application Migration Service (MGN)
AWS Application Migration Service (MGN) is a **lift-and-shift migration tool** that helps move applications from on-premises (or other clouds) to AWS without major modifications. It automates the replication of live systems, ensuring minimal downtime.

**How AWS MGN Works**
* Deploy the AWS MGN agent on each on-premise VM.
* MGN continuously replicates the VM's data (disks, OS, and applications) to AWS.
* You can launch test instances before performing the final cutover.
* *Once verified, switch over to AWS and shut down on-prem VMs.

**Key Features**
* **Continuous Replication** - Minimal downtime migration.
* **Automated Orchestration** - Handles OS updates, networking, and instance types.
* **Supports Any Workload** - Windows, Linux, databases, and custom applications.
* **Rollback Available** - If needed, revert to on-prem operations.

**Step-by-Step Migration Using AWS MGN**
1. Navigate to AWS MGN Console.
2. Configure Replication Settings (AWS region, IAM roles, VPC, etc.).
3. **Download and install the MGN agent on your on-premise VMs.**
4. Start Data Replication
    * **AWS MGN continuously copies your disks to the cloud.**
5. Launch Test Instances (Optional)
6. Verify performance and configurations before cutover.
7.Once satisfied, launch production instances, redirect traffic, and retire on-prem.

### 3. AWS Database Migration Service (AWS DMS)
AWS Database Migration Service (DMS) is a fully managed service that migrates databases to AWS with minimal downtime. It supports homogeneous (same database type) and heterogeneous (different database type) migrations.

**Common Use Cases**
* Lift-and-Shift Migration → Move on-prem databases (Oracle, SQL Server, MySQL, PostgreSQL) to AWS RDS, Aurora, or DynamoDB.
* Cross-Platform Migration → Convert Oracle to PostgreSQL, SQL Server to MySQL, etc.
* **Continuous Data Replication** → Keep on-prem and AWS databases in sync for real-time analytics or disaster recovery.

**How AWS DMS Works**
* Create a Replication Instance
    * **The DMS replication instance runs in AWS and manages data transfer.**
* Define Source & Target Endpoints
    * Source: On-prem database (MySQL, Oracle, SQL Server, etc.).
    * Target: AWS RDS, Aurora, DynamoDB, or another AWS database.
* Choose Migration Type
    * Full Load - One-time copy of all data.
    * Change Data Capture (CDC) - Continuous replication of changes.
    * Both - Full load followed by CDC.
* Start the Migration Task
    * DMS extracts data from the source, transforms if needed, and loads it into the target database.
* Monitor & Optimize
    * Use AWS CloudWatch to track migration progress.
    * Optimize performance with AWS Schema Conversion Tool (SCT) for heterogeneous migrations.

**Key Features of AWS DMS**
* **Minimal Downtime** - Keeps source DB running during migration.
* **Change Data Capture (CDC)** - Syncs only incremental changes.
* **Supports Homogeneous & Heterogeneous Migrations** - Migrate within the same DB type or between different DB types.
* **Schema Conversion Tool (SCT)** - Automatically converts schema for different database engines.
* **Multi-Target Support** - Migrate to multiple destinations like RDS, Redshift, S3, DynamoDB.


### 4. AWS DataSync
AWS DataSync is a managed data transfer service designed to move large volumes of data between on-premises storage and AWS quickly and securely.

Use Cases
* Migrate large datasets to AWS S3, EFS, FSx
* Continuous data replication for hybrid cloud
* Automated backups of on-prem storage to AWS

**How AWS DataSync Works**
* **Install the agent on-prem to connect local storage.**
* Define a DataSync Task
    * Configure source (on-prem NFS/SMB storage) and destination (AWS S3, EFS, or FSx).
* Set Transfer Options
    * Define sync frequency, bandwidth limits, and integrity checks.
* Start Data Transfer
    * DataSync compresses, encrypts, and transfers data efficiently.
* Monitor & Automate
    * Track transfer progress using AWS CloudWatch and schedule future syncs.

**Key Features**
* **High-Speed Transfers** - 10x faster than traditional copy tools.
* **Data Validation & Integrity** - Ensures data is intact after transfer.
* **Incremental Sync** - Transfers only changed data after initial migration.
* **Supports NFS, SMB, Amazon S3, EFS, FSx** - Works with file systems and object storage.

### Comparison table

| Feature | Application Migration Service (MGN)	| Database Migration Service (DMS)	| Application Discovery Service (ADS)	| DataSync |
|---|---|---|---|---|
| Purpose | Lift-and-shift migration of servers	| Migrate & replicate databases	| Discover on-prem apps & dependencies |	Transfer large-scale data to/from AWS |
| Best For|	VM migrations to AWS (EC2)	| Database migrations (homogeneous & heterogeneous)|	Pre-migration planning & dependency mapping	| Moving files, backups, and datasets |
| Type of Data| Full server images (OS, apps, data) | Databases (SQL, NoSQL, etc.) | Server metadata, dependencies, performance stats | Files & object storage|
| Real-time Sync | ✅ Yes (continuous replication) | ✅ Yes (CDC - Change Data Capture)| 	❌ No | ✅ Yes (incremental sync) |
| Automated Cutover | ✅ Yes (minimal downtime) | ✅ Yes (CDC ensures minimal downtime) | ❌ No | ❌ No |
| Pre-Migration Analysis | ❌ No | ❌ No | ✅ Yes | ❌ No |
| Post-Migration Optimization | ❌ No | ✅ Yes (schema conversion, tuning) | ❌ No | ❌ No |
| Supported Sources | VMware, Hyper-V, Physical Servers | Oracle, MySQL, PostgreSQL, SQL Server, MongoDB, etc.| On-premises servers (Windows/Linux)| NFS, SMB, S3, HDFS |
| Supported Destinations | AWS EC2 (rehost strategy) | RDS, Aurora, Redshift, DynamoDB | AWS Migration Hub | Amazon S3, EFS, FSx, On-premises |
| Primary AWS Service Used| AWS MGN| AWS DMS| AWS Migration Hub| AWS DataSync |
| Security & Encryption|	✅ Yes (TLS, IAM)|	✅ Yes (IAM, encryption at rest)|	✅ Yes (IAM)|	✅ Yes (AES-256, TLS)|
| Typical Use Case | Full server migration to AWS| Migrating production databases| Assessing migration impact | Large-scale data transfers | 
| Pricing Model| Based on replication duration | Based on data volume migrated | Free with AWS Migration Hub| Based on data transferred|

**Summary**
* Use MGN when migrating full servers (VMs, physical machines).
* Use DMS when migrating or replicating databases.
* Use ADS to analyze and plan application migrations before executing them.
* Use DataSync to move large datasets between on-premises and AWS.


### Migration Patterns

When migrating from on-premise VMs to AWS, there are multiple migration patterns based on workload type, complexity, and business needs. Here are the top migration patterns you should know:

#### 1. Lift-and-Shift (Rehost)
Best for: **Quick migrations with minimal changes.**
* Move workloads as-is from on-prem VMs to AWS EC2.
* Ideal for legacy applications that are hard to refactor.
* Uses AWS Application Migration Service (MGN) for VM replication.

AWS Tools:
* AWS MGN (for VM replication)
* AWS DataSync (for file migration)
* AWS DMS (for database migration)

Example: Migrate a Windows Server running SQL to AWS EC2 & RDS.

#### Lift-Tinker-and-Shift (Replatform)
Best for: **Minor optimizations during migration**.
* Make small changes to improve efficiency.
* Example: **Moving a database from on-prem SQL Server to Amazon RDS instead of EC2**.
* Can include OS upgrades, moving to managed services, etc.

AWS Tools:
* AWS MGN (for VMs)
* AWS DMS (for moving databases to RDS)
* AWS EFS / FSx (for modernizing storage)

Example: Migrating a monolithic app to AWS and switching its storage from local disks to Amazon EFS for scalability.

#### Refactor (Rearchitect)
Best for: Modernizing applications using cloud-native architecture.
* Break down monolithic apps into microservices.
* Use serverless, containers, and managed services.
* Redesign using AWS Lambda, Amazon ECS/EKS, DynamoDB, S3, etc.

AWS Tools:
* AWS Lambda (for serverless computing)
* Amazon ECS/EKS (for containerizing applications)
* Amazon API Gateway (for API-based architecture)

Example: Moving a legacy e-commerce app to AWS Lambda & API Gateway to remove server dependencies.

#### Retire
Best for: Cleaning up unused workloads.
* Identify unused apps and decommission them.
* Reduces licensing costs and maintenance overhead.
* Use AWS Application Discovery Service to find unused workloads.

AWS Tools:
* AWS Trusted Advisor (for analyzing underutilized resources)
* AWS Application Discovery Service (to map dependencies)

Example: Shutting down a legacy reporting tool that has been replaced by AWS QuickSight.

#### Retain (Hybrid Model)
Best for: Keeping some workloads on-prem while using AWS for others.
* Maintain a hybrid cloud with on-prem & AWS integration.
* **Use AWS Outposts for AWS services on-prem**.
* Ideal for apps requiring low latency or compliance restrictions.

AWS Tools:
* AWS Outposts (AWS infrastructure on-prem)
* AWS Direct Connect (dedicated network link to AWS)
* AWS Storage Gateway (for hybrid storage)

Example: A bank keeps sensitive customer data on-prem but uses AWS for analytics.

#### Relocate
Best for: Moving entire VMware environments to AWS.
* Shift VMware workloads without conversions.
* Fully managed via VMware Cloud on AWS.

🚀 AWS Tools:
* VMware Cloud on AWS

Example: A company using VMware ESXi on-prem moves everything to VMware Cloud on AWS.

#### Choosing the Right Pattern
| Pattern | Complexity | Use Case |
|---|---|---|
| Lift-and-Shift (Rehost) | Low|	Move VMs to AWS without changes |
| Lift-Tinker-and-Shift (Replatform) | Medium|	Optimize during migration (e.g., RDS instead of EC2) |
| Refactor (Rearchitect)|	High	| Modernize with cloud-native technologies |
| Retire	|Low	| Decommission unused workloads |
| Retain (Hybrid)|	Medium |	Keep some workloads on-prem, use AWS for others |
| Relocate	Medium|	Migrate VMware workloads to AWS|



### AWS Outposts
AWS Outposts is a **fully managed service** that extends AWS infrastructure, services, and APIs to your on-premises data center or co-location space. 
It allows you to run AWS services locally while still being able to connect seamlessly with the AWS cloud.

AWS Outposts is designed for:
1. Low-latency Applications
    * Ideal for workloads requiring millisecond latency (e.g., manufacturing, healthcare, gaming).
2. **Data Residency Compliance**
    * Some businesses require their data to remain within a specific country/region.
3. **Hybrid Cloud Strategy**
    * Companies that need to mix on-premises infrastructure with AWS cloud services.
4. Edge Computing
    * Use AWS services closer to end users or devices (e.g., factories, retail stores, telecom).



**How AWS Outposts Works**  
AWS delivers physical racks (like those in AWS data centers) to your location. These racks are fully managed and maintained by AWS.

1. You Get:
    * The same AWS services, APIs, and tools as in the cloud.
    * Compute (EC2), Storage (EBS, S3), Databases (RDS), Networking (VPC, Load Balancer) on-premises.
    * Centralized management via the AWS console.
2. AWS Handles:
    * Installation, patching, monitoring, and updates.
    * Secure connections between Outposts and AWS cloud.

**Types of AWS Outposts**
1. Outposts Racks (Full AWS infrastructure)
    * Pre-configured 42U racks with AWS hardware
    * Fully managed and supports a wide range of AWS services
2. Outposts Servers (Smaller, for edge computing)
    * Standalone 1U or 2U servers
    * Ideal for small sites, branch offices, and edge locations    

**Deployment Steps**
1. Order AWS Outposts via AWS console.
2. AWS delivers & installs it in your data center.
3. Connect it to AWS Region using a dedicated link.
4. Start using AWS services on-premises.

