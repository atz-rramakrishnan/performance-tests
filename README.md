# CPI Performance Test Scripts - Implementation Guide

## Story: PES-2290 - Prepare CPI Log Performance Test Scripts

### Context
Create 2 JMeter test scripts for performance testing CPI logs for Logging 3.0 project:
1. **Gradual Ramp-Up Test**: Gradually increases TPS to maximum load
2. **Sudden Burst Test**: Immediate spike to maximum load

### Endpoint Under Test
- **URL**: `{{baseUrl}}/services/collector/event`
- **Method**: POST
- **Target**: Apigee Logging API (will route through Kirby → PubSub → GCR)

### Test Requirements (from test_plan.md)

#### CPI Log Types to Test:
1. **CPI Log Collector Logs** - Multiple logs in 1 request (mirrors production behavior)
   - Sample: `cpi_log_collector.json`
   - Send multiple log entries per request
   
2. **PubSub Error Logs** - Large payloads with stacktrace/inbounddata
   - Sample: `pubsub_error_log.json`
   - Contains XML data, stacktrace, error details

#### Performance Targets:
- **Maximum TPS**: 5,000 transactions per second
- **Expected Log Volume**: ~20,000 logs end-to-end
- **Test Duration**: TBD based on ramp-up configuration
- **Environment**: QA

---

## Step-by-Step Implementation Plan

### Phase 1: Setup & Analysis ✓

#### Step 1: Understand Existing CAR Test Structure ✓
Review the CAR Load Test JMX file to understand:
- ✓ Thread Group configuration (Get Auth Token groups)
- ✓ HTTP Request structure (Request Auth Token sampler)
- ✓ Authentication flow (OAuth 2.0 token retrieval)
- ✓ Data extraction (JSON Extractor for accesstoken)
- ✓ Listeners configuration (View Results Tree, Summary Report)
- ✓ Constant Timer usage (300000ms = 5min delay)

**Key Observations from CAR Test:**
- Uses standard Thread Group (not Ultimate Thread Group)
- OAuth authentication at the beginning
- Separates different test scenarios into different Thread Groups
- Uses JSR223 PostProcessor for data manipulation
- HTTP Header Manager for content-type and auth headers
- CSV Data Set Config for parameterization

---

### Phase 2: Create Test Infrastructure

#### Step 2: Organize Test Files
```
performance-tests/
├── README.md (this file)
├── cpi-tests/
│   ├── CPI_Logs_Gradual_Ramp.jmx
│   ├── CPI_Logs_Sudden_Burst.jmx
│   ├── test-data/
│   │   ├── cpi_log_collector.json
│   │   ├── pubsub_error_log.json
│   │   └── cpi_log_collector_sample.json
│   └── results/
│       └── .gitkeep
├── api-tests/
│   └── (Reserved for PES-2207)
└── shared/
    └── config.properties
```

**Action Items:**
- [x] Create folder structure
- [ ] Copy sample JSON files to test-data folder
- [ ] Create configuration file for environment variables

#### Step 3: Prepare Test Data Files

**Copy sample files:**
```powershell
# From Downloads to test-data
Copy-Item "C:\Users\rramakrishnan\Downloads\cpi_log_collector.json" -Destination ".\performance-tests\cpi-tests\test-data\"
Copy-Item "C:\Users\rramakrishnan\Downloads\pubsub_error_log.json" -Destination ".\performance-tests\cpi-tests\test-data\"
Copy-Item "C:\Users\rramakrishnan\Downloads\cpi-log-collector-sample.json" -Destination ".\performance-tests\cpi-tests\test-data\"
```

---

### Phase 3: Create Test Script 1 - Gradual Ramp-Up

#### Step 4: Configure Thread Group for Gradual Ramp-Up

**Test Configuration:**
- **Name**: "CPI Logs - Gradual Ramp-Up to 5000 TPS"
- **Thread Configuration**:
  - Number of Threads: 500 (simulates 500 concurrent users)
  - Ramp-up Period: 600 seconds (10 minutes)
  - Loop Count: Infinite (or high number like 1000)
  - Duration: 1800 seconds (30 minutes)
  
**Thread Group Properties:**
- Action after Sampler error: Continue
- Same user on each iteration: Yes
- Delay Thread creation until needed: Yes

**Rationale**: 
- 500 threads ramping up over 10 minutes = ~50 threads/minute
- Each thread making requests will gradually increase load
- Duration of 30 minutes allows for steady-state testing after ramp-up

#### Step 5: Add HTTP Request Samplers

Create 2 HTTP Request Samplers within the Thread Group:

**Sampler 1: CPI Log Collector Request**
- **Name**: "POST CPI Log Collector Logs"
- **Protocol**: https
- **Server Name**: `${__P(baseUrl,api.qa.aritzia.com)}`
- **Port**: 443
- **Method**: POST
- **Path**: `/services/collector/event`
- **Body Data**: Read from `cpi_log_collector.json`
- **Content-Type**: application/json

**Sampler 2: PubSub Error Log Request**
- **Name**: "POST PubSub Error Logs"
- **Protocol**: https
- **Server Name**: `${__P(baseUrl,api.qa.aritzia.com)}`
- **Port**: 443
- **Method**: POST
- **Path**: `/services/collector/event`
- **Body Data**: Read from `pubsub_error_log.json`
- **Content-Type**: application/json

#### Step 6: Add HTTP Header Manager

**Headers to Add:**
```
Content-Type: application/json
Accept: application/json
```

**Note**: Authentication may be required - check with team if Apigee API key needed

#### Step 7: Add Constant Throughput Timer

**Purpose**: Control the request rate to reach 5000 TPS gradually

- **Name**: "Throughput Controller"
- **Target Throughput**: ${__P(targetTPS,5000)} requests/minute
- **Calculate Throughput based on**: all active threads

**Note**: JMeter throughput is in requests per minute, so 5000 TPS = 300,000 requests/minute

#### Step 8: Add Listeners for Result Analysis

Add these listeners to monitor test execution:

1. **View Results Tree** - For debugging during test creation
2. **Summary Report** - Overall statistics
3. **Aggregate Report** - Detailed metrics
4. **Response Time Graph** - Visual representation
5. **Active Threads Over Time** - Thread ramp-up visualization

---

### Phase 4: Create Test Script 2 - Sudden Burst

#### Step 9: Configure Thread Group for Sudden Burst

**Test Configuration:**
- **Name**: "CPI Logs - Sudden Burst to 5000 TPS"
- **Thread Configuration**:
  - Number of Threads: 500
  - Ramp-up Period: 10 seconds (immediate spike)
  - Loop Count: Infinite (or 500)
  - Duration: 900 seconds (15 minutes)
  
**Thread Group Properties:**
- Action after Sampler error: Continue
- Same user on each iteration: Yes
- Delay Thread creation until needed: No (create all threads immediately)

**Rationale**: 
- All 500 threads created in 10 seconds creates immediate load spike
- Tests system's ability to handle sudden traffic bursts
- 15-minute duration allows observation of system behavior under sustained burst

#### Step 10: Replicate Samplers and Configuration

- Copy all HTTP Samplers from Gradual Ramp-Up test
- Copy HTTP Header Manager
- Copy Constant Throughput Timer
- Copy Listeners

**Key Difference**: Only the Thread Group ramp-up time changes (10s vs 600s)

---

### Phase 5: Parameterization & Configuration

#### Step 11: Create Configuration File

**File**: `performance-tests/shared/config.properties`

```properties
# Environment Configuration
baseUrl=api.qa.aritzia.com
targetTPS=5000
testDuration=1800

# Authentication (if needed)
# apiKey=your-api-key-here
# authToken=your-token-here

# Test Data Files
cpiLogCollectorFile=../cpi-tests/test-data/cpi_log_collector.json
pubsubErrorLogFile=../cpi-tests/test-data/pubsub_error_log.json
```

#### Step 12: Add User Defined Variables

In each JMX file, add User Defined Variables at Test Plan level:

Variables to define:
- `baseUrl` = `${__P(baseUrl,api.qa.aritzia.com)}`
- `targetTPS` = `${__P(targetTPS,5000)}`
- `testDuration` = `${__P(testDuration,1800)}`

---

### Phase 6: Testing & Validation

#### Step 13: Local Smoke Test

**Run small-scale test locally:**

```powershell
# Test with 1 thread for 30 seconds
cd C:\Tools\apache-jmeter-5.6.3\bin

# Gradual Ramp-Up Test
.\jmeter.bat -n `
  -t "C:\Users\rramakrishnan\Workspaces\platform-logs-publisher\platform-logs-publisher\performance-tests\cpi-tests\CPI_Logs_Gradual_Ramp.jmx" `
  -l "C:\Users\rramakrishnan\Workspaces\platform-logs-publisher\platform-logs-publisher\performance-tests\cpi-tests\results\gradual-smoke-test.jtl" `
  -JbaseUrl=api.qa.aritzia.com `
  -JtargetTPS=10 `
  -JtestDuration=30

# Sudden Burst Test
.\jmeter.bat -n `
  -t "C:\Users\rramakrishnan\Workspaces\platform-logs-publisher\platform-logs-publisher\performance-tests\cpi-tests\CPI_Logs_Sudden_Burst.jmx" `
  -l "C:\Users\rramakrishnan\Workspaces\platform-logs-publisher\platform-logs-publisher\performance-tests\cpi-tests\results\burst-smoke-test.jtl" `
  -JbaseUrl=api.qa.aritzia.com `
  -JtargetTPS=10 `
  -JtestDuration=30
```

**Validation Checks:**
- [ ] Requests are sent successfully (no connection errors)
- [ ] Response codes are 200 OK (or expected codes)
- [ ] JSON payload is correctly formatted
- [ ] Headers are correctly set
- [ ] Results are logged to .jtl file

#### Step 14: Upload to VM and Test

**Steps:**
1. Copy JMX files to QA VM
2. Copy test-data folder to VM
3. Run reduced-scale test (e.g., 50 threads, 1 minute):
   ```bash
   cd /opt/jmeter/bin
   ./jmeter.sh -n -t CPI_Logs_Gradual_Ramp.jmx -l results.jtl
   ```
4. Verify logs appear in:
   - Apigee (check proxy logs)
   - Kirby metrics
   - PubSub topics
   - GCR logs

---

### Phase 7: Documentation & Handoff

#### Step 15: Create Execution Guide

**File**: `performance-tests/cpi-tests/EXECUTION_GUIDE.md`

Document:
- How to run tests
- Required parameters
- Expected results
- Troubleshooting steps
- Success criteria checklist

#### Step 16: Update Main README

Add section about performance tests to main project README:
```markdown
## Performance Tests

Performance test scripts are located in `/performance-tests` directory.

See [Performance Tests README](performance-tests/README.md) for details.
```

#### Step 17: Commit and Push

```powershell
git add performance-tests/
git commit -m "feat(performance): Add CPI log performance test scripts

- Add gradual ramp-up test script (CPI_Logs_Gradual_Ramp.jmx)
- Add sudden burst test script (CPI_Logs_Sudden_Burst.jmx)
- Add test data samples for CPI log collector and PubSub error logs
- Add comprehensive README with implementation guide
- Add execution guide for running tests

Implements PES-2290: Prepare CPI Log Performance Test Scripts"

git push origin feat/pes-2290/cpi-performance-tests
```

---

## Key Differences: Gradual vs Burst Tests

| Aspect | Gradual Ramp-Up | Sudden Burst |
|--------|----------------|--------------|
| **Ramp-up Time** | 600 seconds (10 min) | 10 seconds |
| **Thread Creation** | Delayed until needed | All created immediately |
| **Duration** | 1800 seconds (30 min) | 900 seconds (15 min) |
| **Purpose** | Test system scaling | Test spike handling |
| **Load Pattern** | Linear increase | Immediate full load |

---

## JMeter Configuration Tips

### Ultimate Thread Group (if needed)
If you want more control over load profile, use Ultimate Thread Group plugin:
- Already installed in your JMeter
- Allows multiple ramp-up/ramp-down stages
- Better for complex load patterns

### Memory Configuration
For high-load tests, increase JMeter heap:
```bash
# Edit jmeter.bat (Windows) or jmeter.sh (Linux)
set HEAP=-Xms2g -Xmx4g -XX:MaxMetaspaceSize=512m
```

### Best Practices
1. **Always test in CLI mode** (non-GUI) for actual performance tests
2. **Use -l flag** to save results to file
3. **Use -e -o flags** to generate HTML reports
4. **Remove View Results Tree** listener for actual tests (high memory usage)
5. **Use CSV for large test data** instead of inline JSON
6. **Parameterize everything** using User Defined Variables

---

## Success Criteria (from test_plan.md)

After running tests, verify:

### GCR Logs
- [ ] No spike in 429/500/503/504 errors
- [ ] Investigate any error spikes

### SLOs
- [ ] GCR Availability Avg >= 99.5%
- [ ] GCR Latency Avg <= 250ms
- [ ] Request Throughput handles 10k TPS successfully

### Datadog Dashboards
- [ ] GCR - Requests & Errors widget shows expected volume
- [ ] GCR CPU & Memory Utilization within acceptable range
- [ ] GCR Instance Count scales appropriately

### PubSub Metrics
- [ ] UnAcked Messages recover to 0 after test
- [ ] Message Send Rate matches expected TPS

---

## Test Execution Checklist

### Before Running Test
- [ ] Coordinate with D&A team (Kirby monitoring)
- [ ] Set test date/time
- [ ] Confirm QA environment is stable
- [ ] Backup current GCR configuration
- [ ] Clear any existing PubSub backlogs

### During Test
- [ ] Monitor Datadog dashboards in real-time
- [ ] Watch for error spikes
- [ ] Track GCR auto-scaling
- [ ] Monitor PubSub queue depth

### After Test
- [ ] Generate JMeter HTML report
- [ ] Export Datadog dashboard snapshots
- [ ] Document any issues observed
- [ ] Compare results with success criteria
- [ ] Share findings with team

---

## Troubleshooting

### Common Issues

**Issue**: Connection timeout errors
- **Solution**: Check firewall, VPN connection, or increase timeout in HTTP Request

**Issue**: 401 Unauthorized responses
- **Solution**: Verify API key/token configuration in HTTP Header Manager

**Issue**: JMeter runs out of memory
- **Solution**: Increase heap size or reduce thread count

**Issue**: Inconsistent throughput
- **Solution**: Add Constant Throughput Timer or reduce target TPS

---

## References

- **Test Plan**: [test_plan.md](../test_plan.md)
- **Story**: PES-2290: Logging 3.0 - Prepare CPI Log Performance Test Scripts
- **Related Story**: PES-2207 (API Log Performance Tests - separate)
- **JMeter Best Practices**: https://jmeter.apache.org/usermanual/best-practices.html
- **Datadog Dashboard**: https://us5.datadoghq.com/dashboard/... (add actual link)

---

## Next Steps

1. ✅ Create branch: `feat/pes-2290/cpi-performance-tests`
2. [ ] Create folder structure and copy sample files (Step 2-3)
3. [ ] Build Gradual Ramp-Up test in JMeter GUI (Steps 4-8)
4. [ ] Build Sudden Burst test in JMeter GUI (Steps 9-10)
5. [ ] Add parameterization (Steps 11-12)
6. [ ] Run local smoke tests (Step 13)
7. [ ] Upload to VM and validate (Step 14)
8. [ ] Create documentation (Steps 15-16)
9. [ ] Commit and push changes (Step 17)
10. [ ] Create Pull Request for review

---

**Created**: January 8, 2026
**Author**: Implementation Guide
**Story**: PES-2290
