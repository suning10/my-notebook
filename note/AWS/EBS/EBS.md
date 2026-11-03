# EBS
- what is ebs
  - elastic block store
    - network drive that can attach to instance
    - they can be mount to one instance at a time
  - it can detach and attach to another quickly
    - **only available to attach to the same AZ
  - it has provisioned size (Size in GB, IOPS(IO per second))
  - one instance can attach multiple EBS
- Delete on Termination 
  - default is it will be deleted

## EBS Snapshot
- Used for backup
  - Create Snapshots
    - can copy from one AZ to another
      - for disaster recovery
  - Create volume from snapshots
    - recover
- certain retention rules
  - move to recycle bin in x days

## EBS Volume Type
- gp2/gp3 (SSD) 
  - general purpose 
  - Dev server
- io1/io2
  - highest performance 
    - critical workload (intensive)
    - 80K IOPS
    - DB
- st (HDD)
  - HDD volume for high throughput 
  - 500 IOPS
  - log/bigdata/Data Warehosue
- scl(HDD)
  - HDD volume for less frequent 
  - 250 IO
- **can only boot use gp/io**

## multi-attach
- only io1/io2 can use 
- attach volume to multiple instance 
- application must handle concurrent Write


## EBS Encryption 
- to encrypt unencrypt volume 
  1. create a snapshot
  2. create a volume from snapshot
  3. check the encrypt box 

## EFS (Elastic File Storage)
-  for share file across different EC2 instances 
- suited for microservice structure that need to access same file
- setting 
  - Elastic
    - recommended
  - Bursting 
    - best performance
- Compare with EBS
  - EBS
    - can only move to another AZ through snapshot
    - loca to one AZ
    - do not allow multi attach (except IO1/2 )
  - EFS
    - 100 instance across AZ
    - good for hosting website that available across regions
    - only have Linux 
    - more expensive 

```python
Need shared file access?          → EFS ✅
Single instance block storage?    → EBS ✅
Store images/videos/backups?      → S3  ✅
High-performance database?        → EBS ✅
Multiple containers sharing data? → EFS ✅
Static website hosting?           → S3  ✅

EFS vs EBS vs S3
Feature	EFS	EBS	S3
Type	Network File System	Block Storage	Object Storage
Access	Multiple instances simultaneously	One instance at a time	Via HTTP/API
Protocol	NFS	iSCSI/Block	REST API
Scalability	Auto scales	Fixed size (manual resize)	Unlimited
Use Case	Shared files	OS disk, DB	Backup, static assets
Persistence	Independent of EC2	Tied to AZ	Independent
Pricing	Pay per GB used	Pay per provisioned GB	Pay per GB stored
```

```js
1. Create EFS File System
         │
2. Create Mount Targets (per AZ)
         │
3. Configure Security Groups
         │
4. Mount on EC2:
   $ sudo mount -t nfs4 \
     fs-xxxxxxxx.efs.us-east-1.amazonaws.com:/ \
     /mnt/efs
         │
5. Read/Write like a normal directory
   $ echo "hello" > /mnt/efs/file.txt
```