# JMeter Script Creation Guide

## Creating CPI Performance Test Scripts

This guide walks you through creating the actual JMeter (.jmx) test scripts using the JMeter GUI.

---

## Prerequisites

✅ JMeter 5.6.3 installed with Ultimate Thread Group plugin
✅ Test data files copied to `test-data/` folder
✅ Understanding of CAR test structure from screenshots

---

## Part 1: Create Gradual Ramp-Up Test

### Step 1: Open JMeter GUI

```powershell
cd C:\Tools\apache-jmeter-5.6.3\bin
.\jmeter.bat
```

### Step 2: Configure Test Plan

1. Click on "Test Plan" in the left tree
2. **Name**: "CPI Logs - Gradual Ramp-Up Test"
3. **Comments**: "Performance test for CPI logs with gradual ramp-up to 5000 TPS"
4. Check **"Run Thread Groups consecutively"** = ✓ CHECKED (OAuth token must be fetched first)

### Step 3: Add User Defined Variables

Right-click on Test Plan → Add → Config Element → User Defined Variables

Add these variables:
| Name | Value |
|------|-------|
| `baseUrl` | `${__P(baseUrl,api.qa.aritzia.com)}` |
| `targetTPS` | `${__P(targetTPS,5000)}` |
| `testDuration` | `${__P(testDuration,1800)}` |
| `clientId` | `${__P(clientId,YOUR_CLIENT_ID)}` |
| `clientSecret` | `${__P(clientSecret,YOUR_CLIENT_SECRET)}` |

**Note**: Replace `YOUR_CLIENT_ID` and `YOUR_CLIENT_SECRET` with actual OAuth credentials or pass them as command-line parameters.

### Step 4: Add OAuth Token Thread Group

Right-click on Test Plan → Add → Threads → Thread Group

**Thread Group Settings:**
- **Name**: "Get Auth Token"
- **Number of Threads (users)**: `1`
- **Ramp-up period (seconds)**: `1`
- **Loop Count**: `1` (only need token once)
- **Action to be taken after a Sampler error**: Continue

**This thread group must run FIRST** to get the OAuth token before the main load test runs.

### Step 5: Add OAuth Token Request

Right-click on "Get Auth Token" Thread Group → Add → Sampler → HTTP Request

**HTTP Request Settings:**
- **Name**: "Request Auth Token"
- **Protocol**: `https`
- **Server Name or IP**: `${baseUrl}`
- **Port Number**: `443`
- **HTTP Request Method**: `POST`
- **Path**: `/oauth2/token`

**Parameters Tab** (not Body Data):

Click "Add" button to add these parameters:
| Name | Value | Encode? | Include Equals? |
|------|-------|---------|-----------------|
| `grant_type` | `client_credentials` | ✓ | ✓ |
| `client_id` | `${clientId}` | ✓ | ✓ |
| `client_secret` | `${clientSecret}` | ✓ | ✓ |

### Step 6: Extract OAuth Token

Right-click on "Request Auth Token" → Add → Post Processors → JSON Extractor

**JSON Extractor Settings:**
- **Name**: "Read Token"
- **Names of created variables**: `accesstoken`
- **JSON Path expressions**: `$.access_token`
- **Match No. (0 for Random)**: `1`
- **Default Values**: `TOKEN_NOT_FOUND`

### Step 7: Store Token in Properties

Right-click on "Request Auth Token" → Add → Post Processors → JSR223 PostProcessor

**JSR223 PostProcessor Settings:**
- **Name**: "Store Token for Other Threads"
- **Script Language**: `groovy`
- **Script**:
```groovy
props.put('accesstoken', vars.get('accesstoken'))
log.info('Access token stored: ' + vars.get('accesstoken'))
```

This makes the token available to all thread groups via `${__P(accesstoken)}`.

### Step 8: Add Main Thread Group

Right-click on Test Plan → Add → Threads → Thread Group

**Thread Group Settings:**
- **Name**: "CPI Log Requests - Gradual Ramp"
- **Number of Threads (users)**: `500`
- **Ramp-up period (seconds)**: `600` (10 minutes)
- **Loop Count**: Check "Infinite" OR set to high number like `1000`
- **Duration (seconds)**: `${testDuration}` or `1800`
- **Startup delay (seconds)**: `0`

**Advanced Options:**
- **Action to be taken after a Sampler error**: Continue
- **Same user on each iteration**: ✓ Checked
- **Delay Thread creation until needed**: ✓ Checked
- **Specify Thread lifetime**: Leave unchecked

### Step 9: Add HTTP Request - CPI Log Collector

Right-click on Thread Group → Add → Sampler → HTTP Request

**HTTP Request Settings:**
- **Name**: "POST CPI Log Collector Logs"
- **Comments**: "Sends multiple CPI log entries in one request"
- **Protocol**: `https`
- **Server Name or IP**: `${baseUrl}`
- **Port Number**: `443`
- **HTTP Request Method**: `POST`
- **Path**: `/services/collector/event`
- **Content encoding**: `UTF-8`

**Body Data Tab:**
1. Select "Body Data" tab
2. Click the folder icon to load from file
3. Select: `test-data/cpi_log_collector.json`
4. OR paste the JSON content directly

**Parameters Tab**: Leave empty (using Body Data instead)

### Step 10: Add HTTP Request - PubSub Error Log

Right-click on Thread Group → Add → Sampler → HTTP Request

**HTTP Request Settings:**
- **Name**: "POST PubSub Error Logs"
- **Comments**: "Sends large payload with stacktrace and error details"
- **Protocol**: `https`
- **Server Name or IP**: `${baseUrl}`
- **Port Number**: `443`
- **HTTP Request Method**: `POST`
- **Path**: `/services/collector/event`
- **Content encoding**: `UTF-8`

**Body Data Tab:**
1. Load from file: `test-data/pubsub_error_log.json`
2. OR paste JSON content directly

### Step 11: Add HTTP Header Manager

Right-click on "CPI Log Requests - Gradual Ramp" Thread Group → Add → Config Element → HTTP Header Manager

**Headers to Add:**
| Name | Value |
|------|-------|
| `Authorization` | `Bearer ${__P(accesstoken)}` |
| `Content-Type` | `application/json` |
| `Accept` | `application/json` |

**Important**: The `Authorization` header uses the OAuth token fetched in Step 4-7.

### Step 12: Add Constant Throughput Timer

Right-click on "CPI Log Requests - Gradual Ramp" Thread Group → Add → Timer → Constant Throughput Timer

**Timer Settings:**
- **Name**: "Throughput Controller"
- **Target throughput (in samples per minute)**: 
  - Calculate: `${targetTPS} * 60` = `${__groovy(${targetTPS} * 60)}`
  - Or hardcode: `300000` (for 5000 TPS)
- **Calculate Throughput based on**: "all active threads"

**Note**: JMeter uses requests per MINUTE, so 5000 TPS = 300,000 req/min

### Step 13: Add Listeners

#### Add View Results Tree (for debugging)
Right-click on Test Plan → Add → Listener → View Results Tree
- **Name**: "View Results Tree"
- Use this only during test creation, remove before actual test

#### Add Aggregate Report
Right-click on Test Plan → Add → Listener → Aggregate Report
- **Name**: "Aggregate Report"
- Check "Write results to file" and specify path

#### Add Summary Report
Right-click on Test Plan → Add → Listener → Summary Report
- **Name**: "Summary Report"

### Step 14: Save the Test

File → Save As → `CPI_Logs_Gradual_Ramp.jmx`

Save to: `performance-tests/cpi-tests/`

---

## Part 2: Create Sudden Burst Test

### Quick Method: Duplicate and Modify

1. Open `CPI_Logs_Gradual_Ramp.jmx`
2. Save As → `CPI_Logs_Sudden_Burst.jmx`
3. Modify the following:

**Test Plan:**
- **Name**: Change to "CPI Logs - Sudden Burst Test"

**Thread Group:**
- **Name**: Change to "CPI Log Requests - Sudden Burst"
- **Ramp-up period**: Change from `600` to `10` seconds
- **Duration**: Change from `1800` to `900` seconds (15 minutes)
- **Delay Thread creation until needed**: UNCHECK this box

Everything else remains the same!

---

## Testing Your Scripts

### Local Smoke Test in GUI

1. Reduce threads to 1-5 for testing
2. Set duration to 30-60 seconds
3. Click the green Play button ▶ in toolbar
4. Watch "View Results Tree" for requests/responses
5. Check for errors

### Validate Before Committing

Checklist:
- [ ] Both HTTP Requests send successfully
- [ ] JSON payloads are properly formatted
- [ ] Headers are correct
- [ ] No connection errors
- [ ] Response codes are 200 (or expected code)
- [ ] Listeners are configured
- [ ] File saved with correct name

### CLI Test (Small Scale)

```powershell
# Test with reduced load
C:\Tools\apache-jmeter-5.6.3\bin\jmeter.bat -n `
  -t ".\performance-tests\cpi-tests\CPI_Logs_Gradual_Ramp.jmx" `
  -l ".\performance-tests\cpi-tests\results\test.jtl" `
  -JbaseUrl=api.qa.aritzia.com `
  -JtargetTPS=10 `
  -JtestDuration=60
```

---

## Advanced: Using Ultimate Thread Group (Optional)

If you want more control over load profile:

### Add Ultimate Thread Group

Right-click on Test Plan → Add → Threads → Ultimate Thread Group

**For Gradual Ramp:**
| Start Threads Count | Initial Delay (sec) | Startup Time (sec) | Hold Load For (sec) | Shutdown Time (sec) |
|---------------------|---------------------|-------------------|---------------------|---------------------|
| 100 | 0 | 120 | 300 | 60 |
| 200 | 0 | 240 | 600 | 60 |
| 200 | 0 | 240 | 900 | 60 |

This creates a stepped ramp-up pattern.

**For Sudden Burst:**
| Start Threads Count | Initial Delay (sec) | Startup Time (sec) | Hold Load For (sec) | Shutdown Time (sec) |
|---------------------|---------------------|-------------------|---------------------|---------------------|
| 500 | 0 | 10 | 900 | 10 |

This creates an immediate spike.

---

## Tips & Best Practices

### During Development
✓ Start with 1 thread to validate requests
✓ Use View Results Tree to debug
✓ Check response data for errors
✓ Gradually increase thread count

### Before Production Test
✓ Remove or disable View Results Tree listener
✓ Save results to file (.jtl)
✓ Use CLI mode (non-GUI) for execution
✓ Coordinate with monitoring team

### Troubleshooting
- **502/503 errors**: Server overloaded or unreachable
- **Connection timeout**: Network issue or wrong URL
- **401 Unauthorized**: Authentication problem
- **400 Bad Request**: Invalid JSON payload

---

## Sample Test Execution Workflow

1. **Create & Debug** (GUI mode):
   ```
   - Build test in GUI
   - Run with 1-5 threads
   - Verify responses
   - Fix any issues
   ```

2. **Smoke Test** (CLI mode):
   ```powershell
   jmeter -n -t test.jmx -l results.jtl -JtargetTPS=10 -JtestDuration=60
   ```

3. **Medium Scale Test** (CLI mode):
   ```powershell
   jmeter -n -t test.jmx -l results.jtl -JtargetTPS=500 -JtestDuration=300
   ```

4. **Full Scale Test** (CLI mode on VM):
   ```bash
   jmeter -n -t test.jmx -l results.jtl -JtargetTPS=5000 -JtestDuration=1800
   ```

---

## Next Steps After Creating Scripts

1. Test locally with small load
2. Commit to git branch
3. Push to remote
4. Copy to QA VM
5. Run medium-scale validation test
6. Schedule full-scale performance test with team
7. Execute test and monitor dashboards
8. Generate and share results

---

## Reference: CAR Test Structure (from screenshots)

From your existing CAR Load Test, we saw:
- **Get Auth Token** thread group with OAuth flow
- **Request Auth Token** HTTP sampler to `/oauth2/token`
- **JSR223 PostProcessor** for token extraction
- **HTTP Header Manager** with auth headers
- **Constant Timer** for delays between requests
- **Multiple Thread Groups** for different test scenarios

For CPI tests, we simplified since:
- No authentication needed (or use API key in header)
- No complex data extraction required
- Focus on throughput and load patterns

---

**Ready to create your scripts?** Follow Steps 1-10 for each test!
