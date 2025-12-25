---
title: "AWS Glue Unable to Connect to MongoDB on EC2 — Simple Fix"
date: 2025-11-18T11:00:00+05:30
draft: false
description: "How we fixed AWS Glue connection timeout issues while connecting to a self-hosted MongoDB running on EC2."
slug: "aws-glue-mongodb-ec2-connection-issue"
tags: ["AWS", "Glue", "MongoDB", "Troubleshooting"]
categories: ["AWS", "Glue"]
featured_image: "/images/DevOps_Future.png"
 
---

## Problem

While working on AWS Glue jobs for our **Customer Data Platform**, we needed to connect Glue to a **MongoDB database hosted on an EC2 instance** in the same AWS account.

Here is a diagram of the use case:
![Use Case Diagram](/images/aws-glue-mongodb.png "AWS Glue to Mongo connectivity")


Even after configuring:
- Correct VPC and subnet  
- Security groups (allowing traffic as per AWS docs)  
- MongoDB (unmanaged) Glue connector  

…the Glue job **kept failing with a connection timeout**.

---

We initially used the **MongoDB (unmanaged) connector** provided by AWS Glue and configured all required networking details.  
Despite everything looking correct, the connection never succeeded.

---

## Resolution

Instead of using the MongoDB-specific connector, we switched to a **Network-type Glue connection**.

Since the **MongoDB URI was already defined inside the Glue script**, only basic network access was required.

### Steps:
1. Go to **AWS Glue → Data Connections**
2. Create a new connection
3. Select **Data source = Network**
4. Configure the same:
   - VPC
   - Subnet
   - Security group (used by MongoDB EC2)

Attach this **Network connection** to the Glue job.

I was able to connect to MongoDB successfully with this approach.

---

## Key Takeaway

If your MongoDB connection string is already handled in the Glue script,  
**using a Network connection is simpler and more reliable** than the MongoDB connector.


---

📧 Questions or feedback: **hello.saurabhoncloud@gmail.com**  
🌐 **https://saurabhoncloud.com**
