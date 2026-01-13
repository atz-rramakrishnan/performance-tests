# Quick Checklist: Add PubSub Error Log Sampler

## ⚠️ Your JMX files are missing the PubSub Error Log sampler!

Both `CPI_Logs_Gradual_Ramp.jmx` and `CPI-Logs-Sudden_Burst-v3.jmx` currently have **only 1 HTTP sampler** but need **2 samplers**.

---

## Quick Fix Steps

### 1. Open JMX File in JMeter GUI
```powershell
cd C:\Tools\apache-jmeter-5.6.3\bin
.\jmeter.bat
```

Open: `CPI_Logs_Gradual_Ramp.jmx`

### 2. Navigate to Thread Group
In left panel, expand:
```
CPI-Logs-GradualRamp-Up-Test
  └── CPI Log Requests - Gradual Ramp (Thread Group)
      └── POST CPI Log Collector Logs ✅ (exists)
      └── POST PubSub Error Logs ❌ (MISSING - add this!)
```

### 3. Add the Missing Sampler

**Right-click** on "CPI Log Requests - Gradual Ramp" Thread Group  
→ Add → Sampler → HTTP Request

**Configure:**
| Field | Value |
|-------|-------|
| Name | `POST PubSub Error Logs` |
| Comments | `Sends large payload with stacktrace and error details` |
| Protocol | `https` |
| Server Name or IP | `${baseUrl}` |
| Port | `443` |
| Method | `POST` |
| Path | `/services/collector/event` |
| Content Encoding | `UTF-8` |

### 4. Add Request Body

1. Click **"Body Data"** tab (not Parameters)
2. Click the **folder icon** 📁
3. Browse to: `test-data/pubsub_error_log.json`
4. Click OK

**OR** paste this content directly:
```
See file: test-data/pubsub_error_log.json
```

### 5. Save the Test

File → Save (Ctrl+S)

### 6. Repeat for Burst Test

Open: `CPI-Logs-Sudden_Burst-v3.jmx`  
Repeat steps 2-5

---

## Verification Checklist

After adding the sampler, verify:

- [ ] Thread Group has **2 HTTP Requests**:
  - [ ] POST CPI Log Collector Logs
  - [ ] POST PubSub Error Logs
- [ ] Both requests use **Body Data** (not Parameters)
- [ ] Both requests point to `/services/collector/event`
- [ ] Both requests use the same **HTTP Header Manager**
- [ ] File saved successfully

---

## Test Your Changes

Run a quick smoke test (1 thread, 30 seconds):

```powershell
cd C:\Tools\apache-jmeter-5.6.3\bin

.\jmeter.bat -n `
  -t "C:\path\to\CPI_Logs_Gradual_Ramp.jmx" `
  -l "results-test.jtl" `
  -JbaseUrl=api.qa.aritzia.com `
  -JtargetTPS=1 `
  -JtestDuration=30
```

Check results - you should see **both types** of requests in the output!

---

## Why Both Samplers?

1. **CPI Log Collector**: Tests multiple log entries in one request (typical CPI logging)
2. **PubSub Error Logs**: Tests large payloads with stacktraces (error scenarios)

Both are needed to properly simulate production CPI log traffic patterns.

---

## Need Help?

Refer to: [JMETER_SCRIPT_GUIDE.md](./JMETER_SCRIPT_GUIDE.md) - Steps 9 & 10
