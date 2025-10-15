---
title: "How We Optimized Our AWS Costs by 50% — Real Lessons from the Field"
date: 2025-10-15T11:45:00Z
description: "A real-world account of how we cut AWS costs in half, with actionable tips and lessons learned."
---

# How We Optimized Our AWS Costs by 50% — Real Lessons from the Field

A few days ago, we noticed a sudden and unexpected spike in our AWS billing while reviewing the AWS Cost Explorer dashboard. To our surprise, services like **Druid, CloudWatch, S3, EC2, and RDS** had all seen a drastic increase in costs compared to the previous two months — large enough to trigger escalations from our finance team.

That’s when we decided to sit down, review every major service, and implement changes to bring our monthly bill back under control.
After several rounds of analysis and optimization, we managed to cut our AWS costs by more than half. 🎯

This post summarizes the key actions we took — and hopefully, these tips help you optimize your AWS environment too.

---

## 1️⃣ Glue DPU Hours

AWS Glue jobs are billed based on DPU hours (Data Processing Units).
We noticed several ETL jobs taking unusually long to execute — and worse, even failed jobs were running for hours before timing out, still incurring full costs.

Our data engineering team reviewed all such jobs, fixed the errors, and optimized the scripts to reduce runtime significantly.
We also started monitoring Glue jobs regularly from the Glue Job Monitoring page to track DPU usage and status trends.

💡 Tip: Evaluate if your workloads can be moved to a less expensive option like Apache Spark on EMR, AWS Batch, or other managed services depending on your use case.

---

## 2️⃣ S3 Storage Costs

One of our Glue jobs was dumping huge amounts of data into an S3 bucket — multiple terabytes that had accumulated silently over time.

To fix this:

- We implemented S3 lifecycle policies to automatically transition or delete old data (e.g., archive log files after 45 days).
- Set up CloudWatch alarms to notify us when a bucket grows beyond a defined size threshold (in GB or TB, based on usage).

This small change helped us save significantly on S3 storage costs.

---

## 3️⃣ CloudWatch Metrics Explosion

We discovered that during Glue job creation, CloudWatch logging and metrics were enabled by default for all jobs — and we had more than 50 of them.

When we checked CloudWatch, we found over 400,000 metrics created during a single job run, which contributed heavily to the cost.

We then:

- Disabled unnecessary metrics and logging for non-critical jobs.
- Retained logs only for debugging periods.

Always review and control your CloudWatch metric and log retention — it’s a quiet but common cost driver.

---

## 4️⃣ No EC2 Savings Plan

Although we’d been using a consistent number and type of EC2 instances for months, we had overlooked activating a Savings Plan or Reserved Instances.

Once we purchased a 1-year Savings Plan matching our usage, we immediately started saving a noticeable percentage of our EC2 costs.

---

## 5️⃣ RDS Storage Type

Our RDS instance was being used primarily for metadata — not for storing critical production data.
However, it was configured with io1 storage, which is significantly more expensive than gp3.

We migrated the storage type from io1 to gp3 and also downsized the instance after observing consistently low CPU and memory usage.
This alone gave us a solid monthly saving.

---

## 6️⃣ Athena Database and Old Data

Our Athena database contained years of old data that was no longer relevant.
Since Athena queries scan all the data in a table, this unnecessary data was increasing our query costs.

We cleaned up historical datasets and partitioned the remaining ones to minimize future scan costs.

---

## 7️⃣ GP2 Volumes in EKS

By default, AWS EKS provisions gp2 volumes — which are roughly 20% more expensive than gp3 and offer lower performance.

We updated our storage classes and templates to use gp3 by default.
It’s a simple change that adds up over time, especially if you have a large cluster.

---

## 8️⃣ Alerts and Budget Monitoring

Finally, we ensured that proper alerts and budgets are in place across our account:

- AWS Budgets for monthly spending thresholds
- EC2 usage alarms
- Service-specific cost anomalies via Cost Anomaly Detection

Catching unexpected behavior early is the easiest way to prevent big billing surprises later.

---

## ✅ Summary

AWS cost optimization is an ongoing process — not a one-time fix.
The key is visibility: knowing where the money is going and why.

We were able to bring our monthly bill down significantly just by tightening a few configurations and cleaning up unused or misconfigured resources.

If you’re managing multiple services across teams, start small — review Glue, S3, CloudWatch, and EC2 first. You might be surprised how much you can save.

## 💬 Have Questions?

Feel free to reach out if you have any questions or want a detailed breakdown of any step.

📧 Email: [hello.saurabhoncloud@gmail.com](mailto:hello.saurabhoncloud@gmail.com)
