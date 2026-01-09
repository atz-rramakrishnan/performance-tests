# 🚀 Quick Start Guide - CPI Performance Tests

## What You Have Now

✅ Complete folder structure  
✅ All documentation files  
✅ Test data samples (3 JSON files)  
✅ Configuration files  
✅ Git branch: `feat/pes-2290/cpi-performance-tests`

## What You Need to Do Next

### Step 1: Read the Guides (15 minutes)

1. **Start here**: [IMPLEMENTATION_SUMMARY.md](./IMPLEMENTATION_SUMMARY.md)
   - Quick overview of everything
   - What's done, what's next

2. **Then read**: [cpi-tests/JMETER_SCRIPT_GUIDE.md](./cpi-tests/JMETER_SCRIPT_GUIDE.md)
   - Step-by-step instructions for creating JMeter scripts
   - Includes screenshots references from CAR test

### Step 2: Create JMeter Scripts (1-2 hours)

#### Script 1: Gradual Ramp-Up

```powershell
# Open JMeter
cd C:\Tools\apache-jmeter-5.6.3\bin
.\jmeter.bat
```

Follow these steps from JMETER_SCRIPT_GUIDE.md:
1. Configure Test Plan
2. Add User Defined Variables
3. Add Thread Group (500 threads, 600s ramp-up)
4. Add HTTP Request for CPI Log Collector
5. Add HTTP Request for PubSub Error Logs  
6. Add HTTP Header Manager
7. Add Constant Throughput Timer
8. Add Listeners
9. Save as: `CPI_Logs_Gradual_Ramp.jmx`

#### Script 2: Sudden Burst

```powershell
# In JMeter: File → Open → CPI_Logs_Gradual_Ramp.jmx
# Then: File → Save As → CPI_Logs_Sudden_Burst.jmx
```

Change only these settings:
- Test Plan name → "CPI Logs - Sudden Burst Test"
- Thread Group ramp-up → `10` seconds (was 600)
- Thread Group duration → `900` seconds (was 1800)
- Uncheck "Delay Thread creation until needed"

### Step 3: Test Locally (30 minutes)

#### In GUI (for debugging):
1. In JMeter, set threads to `1` in Thread Group
2. Set duration to `30` seconds
3. Click green Play button ▶
4. Watch "View Results Tree" for results
5. Fix any errors

#### In CLI (small scale):
```powershell
cd C:\Tools\apache-jmeter-5.6.3\bin

# Test Gradual Ramp
.\jmeter.bat -n `
  -t "C:\Users\rramakrishnan\Workspaces\platform-logs-publisher\platform-logs-publisher\performance-tests\cpi-tests\CPI_Logs_Gradual_Ramp.jmx" `
  -l ".\results-gradual.jtl" `
  -JbaseUrl=api.qa.aritzia.com `
  -JtargetTPS=10 `
  -JtestDuration=60

# Test Sudden Burst
.\jmeter.bat -n `
  -t "C:\Users\rramakrishnan\Workspaces\platform-logs-publisher\platform-logs-publisher\performance-tests\cpi-tests\CPI_Logs_Sudden_Burst.jmx" `
  -l ".\results-burst.jtl" `
  -JbaseUrl=api.qa.aritzia.com `
  -JtargetTPS=10 `
  -JtestDuration=60
```

### Step 4: Commit & Push (10 minutes)

```powershell
cd C:\Users\rramakrishnan\Workspaces\platform-logs-publisher\platform-logs-publisher

# Add your new JMX files and all documentation
git add performance-tests/

# Commit
git commit -m "feat(performance): Add CPI log performance test scripts

- Add gradual ramp-up test (CPI_Logs_Gradual_Ramp.jmx)
- Add sudden burst test (CPI_Logs_Sudden_Burst.jmx)
- Add comprehensive documentation and guides
- Add test data samples (cpi_log_collector, pubsub_error_log)
- Add configuration and execution guides

Implements PES-2290: Prepare CPI Log Performance Test Scripts"

# Push to remote
git push origin feat/pes-2290/cpi-performance-tests
```

### Step 5: Create Pull Request

1. Go to GitHub repository
2. Click "Create Pull Request"
3. Select: `feat/pes-2290/cpi-performance-tests` → `main`
4. Title: "feat(performance): Add CPI log performance test scripts (PES-2290)"
5. Description: Reference the story and what was implemented
6. Request review from team

---

## Files You'll Create

By the end, you should have these in `performance-tests/cpi-tests/`:

```
cpi-tests/
├── CPI_Logs_Gradual_Ramp.jmx     ← YOU CREATE THIS
├── CPI_Logs_Sudden_Burst.jmx     ← YOU CREATE THIS
├── EXECUTION_GUIDE.md             ← Already created
├── JMETER_SCRIPT_GUIDE.md         ← Already created
├── test-data/
│   ├── cpi_log_collector.json     ← Already copied
│   ├── pubsub_error_log.json      ← Already copied
│   └── cpi-log-collector-sample.json ← Already copied
└── results/
    ├── .gitignore                 ← Already created
    └── .gitkeep                   ← Already created
```

---

## ⏱️ Time Estimates

| Task | Time |
|------|------|
| Read documentation | 15-30 min |
| Create Gradual Ramp script | 45-60 min |
| Create Sudden Burst script | 15-30 min |
| Test locally (GUI) | 15 min |
| Test locally (CLI) | 15 min |
| Commit & Push | 10 min |
| **TOTAL** | **~3 hours** |

---

## ✅ Success Checklist

- [ ] Read IMPLEMENTATION_SUMMARY.md
- [ ] Read JMETER_SCRIPT_GUIDE.md
- [ ] Create CPI_Logs_Gradual_Ramp.jmx in JMeter
- [ ] Create CPI_Logs_Sudden_Burst.jmx in JMeter
- [ ] Test in GUI with 1 thread (both scripts)
- [ ] Test in CLI with 10 TPS for 60s (both scripts)
- [ ] Verify no errors in results
- [ ] Add files to git
- [ ] Commit with descriptive message
- [ ] Push to remote branch
- [ ] Create Pull Request
- [ ] Request team review

---

## 🆘 If You Get Stuck

### Can't find JMeter?
```powershell
cd C:\Tools\apache-jmeter-5.6.3\bin
.\jmeter.bat
```

### Can't find test data files?
They're in: `C:\Users\rramakrishnan\Workspaces\platform-logs-publisher\platform-logs-publisher\performance-tests\cpi-tests\test-data\`

### Script won't run?
1. Check "View Results Tree" in JMeter for errors
2. Verify baseUrl is correct: `api.qa.aritzia.com`
3. Check if authentication is needed (add to HTTP Header Manager)
4. Ensure JSON files are properly formatted

### Need help with JMeter?
1. Review CAR test screenshots you shared
2. Check JMETER_SCRIPT_GUIDE.md for detailed steps
3. Start with 1 thread to debug
4. Use "View Results Tree" to see requests/responses

---

## 📞 Resources

### Documentation (in this folder):
- [IMPLEMENTATION_SUMMARY.md](./IMPLEMENTATION_SUMMARY.md) - Overview
- [README.md](./README.md) - Complete implementation guide (400+ lines)
- [cpi-tests/EXECUTION_GUIDE.md](./cpi-tests/EXECUTION_GUIDE.md) - Run tests
- [cpi-tests/JMETER_SCRIPT_GUIDE.md](./cpi-tests/JMETER_SCRIPT_GUIDE.md) - Create scripts

### JMeter Resources:
- Best Practices: https://jmeter.apache.org/usermanual/best-practices.html
- User Manual: https://jmeter.apache.org/usermanual/index.html

### Project Context:
- Story: PES-2290 in Jira
- Test Plan: [../test_plan.md](../test_plan.md)

---

## 🎯 Your Focus Today

**Main Goal**: Create the 2 JMeter .jmx files

**Follow**: JMETER_SCRIPT_GUIDE.md sections:
- Part 1: Create Gradual Ramp-Up Test (Steps 1-10)
- Part 2: Create Sudden Burst Test (Quick duplication & modification)

**Everything else is already done for you!** 🎉

---

Ready? **Start with Step 1** above! 💪
