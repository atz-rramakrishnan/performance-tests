# QA Load Test Execution Guide - PES-2289

**Project:** atz-api-logs-qa  
**Date:** January 19, 2026  
**Objective:** Execute 1M message backlog load test in QA environment  

---

## Prerequisites

✅ Ensure you have:
- Access to GCP Cloud Shell (or `gcloud` CLI installed locally)
- Access to `atz-api-logs-qa` project
- Permissions to modify Pub/Sub subscriptions
- Access to Cloud Run and Logs Explorer consoles

💡 **Cloud Shell Users:** All commands are optimized for bash. No installation needed!

---

## Quick Reference

### QA Environment Details
- **Project:** `atz-api-logs-qa`
- **Service Account:** `gcp-logs-pubsub-push@atz-api-logs-qa.iam.gserviceaccount.com`
- **Cloud Run URL:** `https://logs-publisher-<HASH>.run.app` (get from console)
- **Subscriptions:** 
  - `logs-apigee-gcr-datadog`
  - `logs-apigee-gcr-gcp`
  - `logs-cpi-gcr-datadog`
  - `logs-cpi-gcr-gcp`

---

## Phase 1: Pre-Test Preparation (10 minutes)

### Step 1.1: Get Current Cloud Run URL
```bash
# Get the Cloud Run service URL
gcloud run services describe logs-publisher \
  --region=us-east4 \
  --project=atz-api-logs-qa \
  --format="value(status.url)"
```

**Expected Output:** `https://logs-publisher-XXXXXXXXX.run.app`

📝 **ACTION:** Copy this URL - you'll need it for reverting back to push mode later.

**Save it here:**
```
QA_CLOUD_RUN_URL=https://logs-publisher-_____________.run.app
```

---

### Step 1.2: Document Current Subscription State
```bash
# List all subscriptions
gcloud pubsub subscriptions list \
  --project=atz-api-logs-qa \
  --format="table(name,pushConfig.pushEndpoint)"
```

**Expected Output:** All 4 subscriptions should show push endpoints with the Cloud Run URL.

📝 **ACTION:** Take a screenshot or copy the output for your records.

---

### Step 1.3: Check Current Message Backlog
```bash
# Check current undelivered messages for all subscriptions
gcloud pubsub subscriptions list \
  --project=atz-api-logs-qa \
  --format="table(name,numUndeliveredMessages)"
```

**Expected Output:** Should show low numbers (< 1000)

📝 **ACTION:** Note the baseline message counts.

---

## Phase 2: Convert Push to Pull Mode (5 minutes)

### Step 2.1: Update logs-apigee-gcr-datadog to Pull
```bash
gcloud pubsub subscriptions update logs-apigee-gcr-datadog \
  --push-endpoint="" \
  --project=atz-api-logs-qa
```

**Expected Output:** 
```
Updated subscription [projects/atz-api-logs-qa/subscriptions/logs-apigee-gcr-datadog].
```

---

### Step 2.2: Update logs-apigee-gcr-gcp to Pull
```bash
gcloud pubsub subscriptions update logs-apigee-gcr-gcp \
  --push-endpoint="" \
  --project=atz-api-logs-qa
```

**Expected Output:** 
```
Updated subscription [projects/atz-api-logs-qa/subscriptions/logs-apigee-gcr-gcp].
```

---

### Step 2.3: Update logs-cpi-gcr-datadog to Pull
```bash
gcloud pubsub subscriptions update logs-cpi-gcr-datadog \
  --push-endpoint="" \
  --project=atz-api-logs-qa
```

**Expected Output:** 
```
Updated subscription [projects/atz-api-logs-qa/subscriptions/logs-cpi-gcr-datadog].
```

---

### Step 2.4: Update logs-cpi-gcr-gcp to Pull
```bash
gcloud pubsub subscriptions update logs-cpi-gcr-gcp \
  --push-endpoint="" \
  --project=atz-api-logs-qa
```

**Expected Output:** 
```
Updated subscription [projects/atz-api-logs-qa/subscriptions/logs-cpi-gcr-gcp].
```

---

### Step 2.5: Verify Pull Mode is Active
```bash
# Verify all subscriptions are in pull mode (pushEndpoint should be empty)
gcloud pubsub subscriptions describe logs-apigee-gcr-datadog \
  --project=atz-api-logs-qa \
  --format="value(pushConfig.pushEndpoint)"

gcloud pubsub subscriptions describe logs-apigee-gcr-gcp \
  --project=atz-api-logs-qa \
  --format="value(pushConfig.pushEndpoint)"

gcloud pubsub subscriptions describe logs-cpi-gcr-datadog \
  --project=atz-api-logs-qa \
  --format="value(pushConfig.pushEndpoint)"

gcloud pubsub subscriptions describe logs-cpi-gcr-gcp \
  --project=atz-api-logs-qa \
  --format="value(pushConfig.pushEndpoint)"
```

**Expected Output:** All commands should return empty (blank lines).

✅ **CHECKPOINT:** If any subscription still shows a push endpoint, repeat that step.

---

## Phase 3: Monitor Message Accumulation (60-90 minutes)

### Step 3.1: Start Monitoring Script

**Bash Monitoring (Cloud Shell)**
```bash
# Run this in Cloud Shell - press Ctrl+C to stop
while true; do
    clear
    echo "=== QA Load Test Message Accumulation ==="
    echo "Time: $(date '+%Y-%m-%d %H:%M:%S')"
    echo ""
    
    gcloud pubsub subscriptions list \
      --project=atz-api-logs-qa \
      --format="table(name,numUndeliveredMessages,oldestUnackedMessageAge)"
    
    echo ""
    echo "Refreshing in 60 seconds... (Press Ctrl+C to stop)"
    sleep 60
done
```

---

### Step 3.2: Wait for Target Backlog

**Target:** ~1,000,000 total messages across all 4 subscriptions

📊 **Progress Tracking:**

| Time Elapsed | Target Messages | Status |
|--------------|-----------------|--------|
| 0 min | Baseline (~0-1000) | ⬜ |
| 15 min | ~250,000 | ⬜ |
| 30 min | ~500,000 | ⬜ |
| 45 min | ~750,000 | ⬜ |
| 60 min | ~1,000,000 | ⬜ |

📝 **ACTION:** Check the boxes above as you reach each milestone.

---

### Step 3.3: Document Peak Backlog State

**When you reach ~1M messages:**
```bash
# Capture final pre-test state
gcloud pubsub subscriptions list \
  --project=atz-api-logs-qa \
  --format="table(name,numUndeliveredMessages,oldestUnackedMessageAge)" \
  > qa-load-test-baseline.txt
```

📝 **ACTION:** Save the output file for your test report.

---

## Phase 4: Execute Load Test - Revert to Push Mode (5 minutes)

⚠️ **IMPORTANT:** This will trigger the flood of messages to Cloud Run. Ensure monitoring is ready!

### Step 4.1: Open Monitoring Dashboards

Before executing, open these in your browser:

1. **Cloud Run Metrics:**
   ```
   https://console.cloud.google.com/run/detail/us-east4/logs-publisher/metrics?project=atz-api-logs-qa
   ```

2. **Logs Explorer:**
   ```
   https://console.cloud.google.com/logs/query?project=atz-api-logs-qa
   ```

---

### Step 4.2: Revert logs-apigee-gcr-datadog to Push

⚠️ **Replace `<HASH>` with the URL you saved in Step 1.1**

```bash
gcloud pubsub subscriptions update logs-apigee-gcr-datadog \
  --push-endpoint="https://logs-publisher-<HASH>.run.app?destination=datadog" \
  --push-auth-service-account="gcp-logs-pubsub-push@atz-api-logs-qa.iam.gserviceaccount.com" \
  --project=atz-api-logs-qa
```

**Expected Output:** 
```
Updated subscription [projects/atz-api-logs-qa/subscriptions/logs-apigee-gcr-datadog].
```

---

### Step 4.3: Revert logs-apigee-gcr-gcp to Push

```bash
gcloud pubsub subscriptions update logs-apigee-gcr-gcp \
  --push-endpoint="https://logs-publisher-<HASH>.run.app?destination=gcp" \
  --push-auth-service-account="gcp-logs-pubsub-push@atz-api-logs-qa.iam.gserviceaccount.com" \
  --project=atz-api-logs-qa
```

**Expected Output:** 
```
Updated subscription [projects/atz-api-logs-qa/subscriptions/logs-apigee-gcr-gcp].
```

---

### Step 4.4: Revert logs-cpi-gcr-datadog to Push

```bash
gcloud pubsub subscriptions update logs-cpi-gcr-datadog \
  --push-endpoint="https://logs-publisher-<HASH>.run.app?destination=datadog" \
  --push-auth-service-account="gcp-logs-pubsub-push@atz-api-logs-qa.iam.gserviceaccount.com" \
  --project=atz-api-logs-qa
```

**Expected Output:** 
```
Updated subscription [projects/atz-api-logs-qa/subscriptions/logs-cpi-gcr-datadog].
```

---

### Step 4.5: Revert logs-cpi-gcr-gcp to Push

```bash
gcloud pubsub subscriptions update logs-cpi-gcr-gcp \
  --push-endpoint="https://logs-publisher-<HASH>.run.app?destination=gcp" \
  --push-auth-service-account="gcp-logs-pubsub-push@atz-api-logs-qa.iam.gserviceaccount.com" \
  --project=atz-api-logs-qa
```

**Expected Output:** 
```
Updated subscription [projects/atz-api-logs-qa/subscriptions/logs-cpi-gcr-gcp].
```

---

### Step 4.6: Verify Push Mode is Active

```bash
# Verify all subscriptions are back in push mode
gcloud pubsub subscriptions list \
  --project=atz-api-logs-qa \
  --format="table(name,pushConfig.pushEndpoint)"
```

**Expected Output:** All 4 subscriptions should show the Cloud Run push endpoint.

✅ **CHECKPOINT:** Load test has now started! Messages are flooding Cloud Run.

📝 **ACTION:** Note the exact time you completed this step: ___________

---

## Phase 5: Real-Time Monitoring (30-60 minutes)

### Step 5.1: Monitor Message Processing Rate

```bash
# Run this in Cloud Shell - press Ctrl+C to stop
while true; do
    clear
    echo "=== QA Load Test - Message Processing ==="
    echo "Time: $(date '+%Y-%m-%d %H:%M:%S')"
    echo ""
    
    # Get subscription data
    gcloud pubsub subscriptions list \
      --project=atz-api-logs-qa \
      --format="table(name,numUndeliveredMessages)"
    
    # Calculate total
    total=$(gcloud pubsub subscriptions list \
      --project=atz-api-logs-qa \
      --format="value(numUndeliveredMessages)" | \
      awk '{sum+=$1} END {print sum}')
    
    echo ""
    echo "Total Undelivered: $total messages"
    echo ""
    echo "Refreshing in 30 seconds... (Press Ctrl+C to stop)"
    sleep 30
done
```

---

### Step 5.2: Monitor Cloud Run Instance Count

```bash
# Run this in a new Cloud Shell tab - press Ctrl+C to stop
while true; do
    clear
    echo "=== Cloud Run Service Status ==="
    echo "Time: $(date '+%Y-%m-%d %H:%M:%S')"
    echo ""
    
    gcloud run services describe logs-publisher \
      --region=us-east4 \
      --project=atz-api-logs-qa \
      --format="table(status.traffic[0].revisionName,status.traffic[0].percent,metadata.annotations.autoscaling.knative.dev/maxScale)"
    
    echo ""
    echo "Refreshing in 30 seconds... (Press Ctrl+C to stop)"
    sleep 30
done
```

---

### Step 5.3: Watch for Errors in Logs

```bash
# Monitor real-time logs for errors
gcloud run services logs tail logs-publisher \
  --region=us-east4 \
  --project=atz-api-logs-qa \
  --format="table(timestamp,severity,textPayload)"
```

---

### Step 5.4: Track Key Metrics in Console

📊 **Monitor these metrics in Cloud Run console:**

| Metric | Target | Status |
|--------|--------|--------|
| Request Latency (p99) | ≤ 250ms | ⬜ |
| Error Rate | < 0.5% | ⬜ |
| CPU Utilization | < 80% | ⬜ |
| Memory Utilization | < 80% | ⬜ |
| Active Instances | Auto-scaling working | ⬜ |
| Request Count | High throughput | ⬜ |

📝 **ACTION:** Check each metric every 5-10 minutes and mark status.

---

## Phase 6: Data Collection & Analysis (15 minutes)

### Step 6.1: Wait for Backlog to Clear

**Target:** All subscriptions should have < 1000 undelivered messages

```bash
# Check if backlog is cleared
gcloud pubsub subscriptions list \
  --project=atz-api-logs-qa \
  --format="table(name,numUndeliveredMessages)"
```

📝 **ACTION:** Note the time when backlog cleared: ___________

---

### Step 6.2: Calculate Total Processing Time

**Start Time (from Phase 4):** ___________  
**End Time (backlog cleared):** ___________  
**Total Duration:** ___________

---

### Step 6.3: Collect Final Metrics

```bash
# Export metrics for reporting
gcloud run services describe logs-publisher \
  --region=us-east4 \
  --project=atz-api-logs-qa \
  --format="json" > qa-load-test-final-state.json
```

---

### Step 6.4: Monitor Errors via Datadog Dashboard

**Primary Method - Datadog Dashboard:**

Navigate to: [Platform - API Logging Dashboard](https://us5.datadoghq.com/dashboard/c6z-wn5-hc2/platform-api-logging)

- Check the **"Platform - API → GCR Errors"** tile
- Set time range to match your test window
- Verify error counts and patterns

📝 **Record error metrics:**
- Total Errors: ___________
- Error Types: ___________

**Alternative - Manual Query (if needed):**

In Logs Explorer, run this query:
```
resource.type="cloud_run_revision"
resource.labels.service_name="logs-publisher"
severity>=ERROR
timestamp>="[START_TIME]"
```

📝 **ACTION:** Document any unexpected errors or anomalies.

---

### Step 6.5: Verify Data Delivery & Compare Logging 2.0 vs 3.0

**Compare Log Counts (PES-2258 Acceptance Criteria):**

Navigate to **GCP Console > Log Analytics > Saved Queries** (atz-api-logs-qa)

**1. Check Logging 3.0 Counts:**

Run these saved queries for the test time range:
- **Logging 3.0 - API Count** (Apigee logs)
- **Logging 3.0 - CPI Count** (CPI logs)

📝 **Record counts:**
- Apigee 3.0 Count: ___________
- CPI 3.0 Count: ___________

**2. Compare Against Logging 2.0:**

Query Logging 2.0 counts for the same time range:
```
resource.type="cloud_run_revision"
resource.labels.service_name="logs-publisher"
jsonPayload.loggingVersion="2.0"
```

📝 **Record counts:**
- Apigee 2.0 Count: ___________
- CPI 2.0 Count: ___________

**3. Check for Duplicates:**

Run these saved queries:
- **Duplicate 3.0 Apigee Logs**
- **Duplicate 3.0 CPI Logs**

📝 **Record duplicates:**
- Apigee Duplicates: ___________
- CPI Duplicates: ___________

**4. Verify Datadog Delivery:**

Check Datadog Logs Explorer:
- Filter: `env:qa` + test time range
- Verify ~50% of messages reached Datadog destination

📝 **Record count:**
- Datadog Log Count: ___________

**✅ Acceptance:**
- 2.0 and 3.0 counts should match closely (within 1-2%)
- Minimal or no duplicate logs
- ~50% delivered to GCP, ~50% to Datadog

---

## Phase 7: Success Criteria Validation ✅

### Step 7.1: Availability Check

✅ **Target:** 99.5% or higher

**Formula:** 
```
Availability = (Total Requests - Failed Requests) / Total Requests * 100
```

**Calculation:**
- Total Requests: ___________
- Failed Requests: ___________
- Availability: ___________% 

**Status:** ⬜ PASS / ⬜ FAIL

---

### Step 7.2: Latency Check

✅ **Target:** p99 ≤ 250ms

**From Cloud Run Metrics:**
- p50 Latency: ___________ms
- p95 Latency: ___________ms
- p99 Latency: ___________ms

**Status:** ⬜ PASS / ⬜ FAIL

---

### Step 7.3: Throughput Check

✅ **Target:** Support 10,000 TPS (Transactions Per Second)

**Peak TPS Observed:** ___________

**Status:** ⬜ PASS / ⬜ FAIL

---

### Step 7.4: Error Rate Check

✅ **Target:** < 0.5% error rate

**Calculation:**
- Total Errors: ___________
- Total Requests: ___________
- Error Rate: ___________% 

**Status:** ⬜ PASS / ⬜ FAIL

---

### Step 7.5: Resource Utilization Check

✅ **Target:** CPU and Memory < 80% sustained

**Observed:**
- Peak CPU: ___________% 
- Peak Memory: ___________% 

**Status:** ⬜ PASS / ⬜ FAIL

---

## Phase 8: Cleanup & Documentation (10 minutes)

### Step 8.1: Verify Normal Operations

```bash
# Confirm subscriptions are in push mode
gcloud pubsub subscriptions list \
  --project=atz-api-logs-qa \
  --format="table(name,pushConfig.pushEndpoint,numUndeliveredMessages)"
```

**Expected:** All subscriptions have push endpoints and low undelivered message counts.

---

### Step 8.2: Update JIRA Story

**PES-2289 Update:**

```markdown
## QA Load Test Results

**Test Date:** [INSERT DATE]
**Environment:** QA (atz-api-logs-qa)
**Messages Processed:** ~1,000,000

### Results Summary:
- Availability: [X]%
- p99 Latency: [X]ms
- Peak TPS: [X]
- Error Rate: [X]%
- Processing Duration: [X] minutes

### Success Criteria:
- [ ] Availability ≥ 99.5%
- [ ] p99 Latency ≤ 250ms
- [ ] Throughput ≥ 10k TPS
- [ ] Error Rate < 0.5%
- [ ] Resource Utilization < 80%

### Issues Found:
1. [List any issues discovered]

### Recommendations:
1. [List any recommendations]
```

---

### Step 8.3: Create Follow-Up Tickets (if needed)

**If any issues were discovered:**

```markdown
Title: [Issue Description] - PES-XXXX

Description:
During QA load test (PES-2289), the following issue was observed:

**Issue:** [Description]
**Impact:** [Severity/Impact]
**Reproduction:** [Steps to reproduce]
**Metrics:** [Relevant metrics/logs]

**Suggested Fix:** [If known]
```

---

## Quick Start Commands (Copy-Paste Ready)

### Convert to Pull Mode (All 4 Commands):
```bash
gcloud pubsub subscriptions update logs-apigee-gcr-datadog --push-endpoint="" --project=atz-api-logs-qa && \
gcloud pubsub subscriptions update logs-apigee-gcr-gcp --push-endpoint="" --project=atz-api-logs-qa && \
gcloud pubsub subscriptions update logs-cpi-gcr-datadog --push-endpoint="" --project=atz-api-logs-qa && \
gcloud pubsub subscriptions update logs-cpi-gcr-gcp --push-endpoint="" --project=atz-api-logs-qa
```

---

### Revert to Push Mode (All 4 Commands):
⚠️ **REPLACE `<HASH>` with your Cloud Run URL hash!**

```bash
gcloud pubsub subscriptions update logs-apigee-gcr-datadog \
  --push-endpoint="https://logs-publisher-<HASH>.run.app?destination=datadog" \
  --push-auth-service-account="gcp-logs-pubsub-push@atz-api-logs-qa.iam.gserviceaccount.com" \
  --project=atz-api-logs-qa && \
gcloud pubsub subscriptions update logs-apigee-gcr-gcp \
  --push-endpoint="https://logs-publisher-<HASH>.run.app?destination=gcp" \
  --push-auth-service-account="gcp-logs-pubsub-push@atz-api-logs-qa.iam.gserviceaccount.com" \
  --project=atz-api-logs-qa && \
gcloud pubsub subscriptions update logs-cpi-gcr-datadog \
  --push-endpoint="https://logs-publisher-<HASH>.run.app?destination=datadog" \
  --push-auth-service-account="gcp-logs-pubsub-push@atz-api-logs-qa.iam.gserviceaccount.com" \
  --project=atz-api-logs-qa && \
gcloud pubsub subscriptions update logs-cpi-gcr-gcp \
  --push-endpoint="https://logs-publisher-<HASH>.run.app?destination=gcp" \
  --push-auth-service-account="gcp-logs-pubsub-push@atz-api-logs-qa.iam.gserviceaccount.com" \
  --project=atz-api-logs-qa
```

---

## Troubleshooting

### Issue: "Permission Denied" Error
**Solution:** Ensure you have the following roles:
```bash
# Check your permissions
gcloud projects get-iam-policy atz-api-logs-qa \
  --flatten="bindings[].members" \
  --filter="bindings.members:user:YOUR_EMAIL"
```

**Required Roles:**
- `roles/pubsub.admin` or `roles/pubsub.editor`
- `roles/run.viewer`

---

### Issue: Subscriptions Not Showing in List
**Solution:** Verify project context:
```bash
gcloud config get-value project
# Should output: atz-api-logs-qa

# If not, set it:
gcloud config set project atz-api-logs-qa
```

---

### Issue: Cannot Update Subscription
**Solution:** Check if subscription exists and is not deleted:
```bash
gcloud pubsub subscriptions describe logs-apigee-gcr-datadog \
  --project=atz-api-logs-qa
```

---

### Issue: Messages Not Accumulating
**Solution:** Verify upstream log generation is active. Check topics:
```bash
gcloud pubsub topics list --project=atz-api-logs-qa
```

---

### Issue: Cloud Run Instances Not Scaling
**Solution:** Check autoscaling settings:
```bash
gcloud run services describe logs-publisher \
  --region=us-east4 \
  --project=atz-api-logs-qa \
  --format="value(metadata.annotations)"
```

---

## Notes

- **Estimated Total Time:** 2-3 hours
- **Best Time to Execute:** During business hours for immediate support if needed
- **Notification:** Inform the team before starting the test
- **Rollback:** If critical issues occur, subscriptions are already in push mode - no rollback needed

---

## Contact Information

**If you encounter issues:**
- Slack: #platform-engineering
- On-call: [Insert on-call contact]
- Documentation: See README.md in this repository

---

**Good luck with your load test! 🚀**
