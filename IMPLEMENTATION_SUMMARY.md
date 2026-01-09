# Story Implementation Summary: PES-2290

## ✅ What's Been Completed

### 1. Branch Setup
- ✅ Created branch: `feat/pes-2290/cpi-performance-tests`
- ✅ Based on latest `main` branch

### 2. Folder Structure Created
```
performance-tests/
├── README.md                          ✅ Complete implementation guide
├── cpi-tests/
│   ├── EXECUTION_GUIDE.md            ✅ How to run tests
│   ├── JMETER_SCRIPT_GUIDE.md        ✅ Step-by-step JMeter creation
│   ├── test-data/
│   │   ├── cpi_log_collector.json     ✅ Copied from Downloads
│   │   ├── pubsub_error_log.json      ✅ Copied from Downloads
│   │   └── cpi-log-collector-sample.json ✅ Copied from Downloads
│   └── results/
│       ├── .gitignore                 ✅ Ignore test results
│       └── .gitkeep                   ✅ Keep directory
├── api-tests/                         ✅ Reserved for PES-2207
└── shared/
    └── config.properties              ✅ Environment configuration
```

### 3. Documentation Created
- ✅ **README.md**: Comprehensive 400+ line implementation guide
- ✅ **EXECUTION_GUIDE.md**: How to run tests (CLI commands, parameters)
- ✅ **JMETER_SCRIPT_GUIDE.md**: Step-by-step JMeter GUI instructions
- ✅ **config.properties**: Environment configuration template

### 4. Test Data
- ✅ All 3 sample JSON files copied to `test-data/`
- ✅ Files are ready to be referenced in JMeter scripts

---

## 🎯 Next Steps (What You Need to Do)

### Immediate (Today):
1. **Review the Documentation**
   - Read [performance-tests/README.md](./README.md) - Main guide
   - Read [JMETER_SCRIPT_GUIDE.md](./cpi-tests/JMETER_SCRIPT_GUIDE.md) - Script creation steps

2. **Open JMeter and Create Scripts**
   - Follow JMETER_SCRIPT_GUIDE.md Step 1-10
   - Create `CPI_Logs_Gradual_Ramp.jmx`
   - Create `CPI_Logs_Sudden_Burst.jmx`
   - Save both to `performance-tests/cpi-tests/`

3. **Test Locally (Small Scale)**
   - Run in GUI with 1-5 threads
   - Validate requests work
   - Check for errors

### This Week:
4. **Run CLI Smoke Test**
   - Use commands from EXECUTION_GUIDE.md
   - Test with 10 TPS for 60 seconds
   - Verify results

5. **Commit Your Work**
   ```powershell
   git add performance-tests/
   git commit -m "feat(performance): Add CPI log performance test scripts"
   git push origin feat/pes-2290/cpi-performance-tests
   ```

6. **Create Pull Request**
   - Open PR from your branch to main
   - Link to PES-2290 story
   - Request team review

### Next Sprint:
7. **Upload to QA VM**
   - Copy JMX files and test-data folder
   - Run medium-scale test (50 threads, 5 minutes)
   - Validate end-to-end flow

8. **Schedule Full Performance Test**
   - Coordinate with D&A team (Kirby monitoring)
   - Set date/time
   - Prepare monitoring dashboards

9. **Execute Performance Test**
   - Run both tests (Gradual & Burst)
   - Monitor Datadog dashboards
   - Document results

10. **Complete Story**
    - Generate HTML reports
    - Document findings
    - Update Confluence
    - Mark PES-2290 as Done

---

## 📋 Implementation Checklist

### Phase 1: Setup ✅
- [x] Create git branch
- [x] Create folder structure
- [x] Copy sample JSON files
- [x] Create documentation
- [x] Create configuration files

### Phase 2: JMeter Scripts (YOU DO THIS)
- [ ] Open JMeter GUI
- [ ] Create Gradual Ramp-Up test (follow JMETER_SCRIPT_GUIDE.md)
- [ ] Create Sudden Burst test
- [ ] Test in GUI with low thread count
- [ ] Save both .jmx files

### Phase 3: Validation
- [ ] Run CLI smoke test (Gradual)
- [ ] Run CLI smoke test (Burst)
- [ ] Verify no connection errors
- [ ] Verify correct payloads sent

### Phase 4: Git & Review
- [ ] Add files to git
- [ ] Commit with descriptive message
- [ ] Push to remote branch
- [ ] Create Pull Request
- [ ] Address review comments

### Phase 5: VM Testing
- [ ] Copy files to QA VM
- [ ] Run medium-scale test
- [ ] Validate logs appear in GCR
- [ ] Confirm PubSub metrics

### Phase 6: Execution
- [ ] Schedule test with team
- [ ] Run full-scale Gradual test
- [ ] Run full-scale Burst test
- [ ] Monitor dashboards
- [ ] Generate reports

### Phase 7: Completion
- [ ] Document results
- [ ] Share with team
- [ ] Update test_plan.md if needed
- [ ] Mark story as Done

---

## 🔑 Key Information

### Test Configuration

#### Gradual Ramp-Up Test
- **Threads**: 500
- **Ramp-up**: 600 seconds (10 minutes)
- **Duration**: 1800 seconds (30 minutes)
- **Target TPS**: 5000
- **Purpose**: Test system scaling capabilities

#### Sudden Burst Test
- **Threads**: 500
- **Ramp-up**: 10 seconds
- **Duration**: 900 seconds (15 minutes)
- **Target TPS**: 5000
- **Purpose**: Test spike handling

### Endpoint Details
- **URL**: `https://api.qa.aritzia.com/services/collector/event`
- **Method**: POST
- **Content-Type**: application/json
- **Authentication**: TBD (check with team)

### Sample Payloads
1. **CPI Log Collector**: Multiple logs per request (~84 lines)
2. **PubSub Error Logs**: Large payload with stacktrace (~67 lines)

---

## 📚 Reference Documents

### In This Repo:
- [Main README](./README.md) - Complete implementation guide
- [Execution Guide](./cpi-tests/EXECUTION_GUIDE.md) - How to run tests
- [JMeter Guide](./cpi-tests/JMETER_SCRIPT_GUIDE.md) - Script creation steps
- [test_plan.md](../test_plan.md) - Original performance test plan

### External:
- JMeter Best Practices: https://jmeter.apache.org/usermanual/best-practices.html
- Story: PES-2290 in Jira
- Related: PES-2207 (API tests - separate story)

---

## 💡 Tips for Success

### Creating JMeter Scripts:
1. **Start Simple**: Begin with 1 thread, verify it works
2. **Use Variables**: Parameterize everything (${baseUrl}, ${targetTPS})
3. **Test Incrementally**: 1 thread → 10 threads → 100 threads → full load
4. **Watch for Errors**: Use View Results Tree during development
5. **Remove Listeners**: Disable heavy listeners before production test

### Running Tests:
1. **Always use CLI mode** for actual performance tests (not GUI)
2. **Save results to file**: Use -l flag with .jtl file
3. **Generate HTML reports**: Use -e -o flags
4. **Monitor in real-time**: Have Datadog dashboards ready
5. **Coordinate with team**: D&A needs to monitor Kirby

### Common Pitfalls to Avoid:
- ❌ Running high-load test in GUI mode (causes memory issues)
- ❌ Forgetting to parameterize configuration
- ❌ Not testing small-scale first
- ❌ Not saving results to file
- ❌ Leaving View Results Tree enabled in production test

---

## 🤝 Team Coordination

### Before Test Execution:
- **Notify**: D&A team (for Kirby monitoring)
- **Schedule**: Pick QA downtime window
- **Prepare**: Clear PubSub backlogs
- **Backup**: Current GCR configuration

### During Test:
- **Monitor**: Datadog dashboards
- **Watch**: Error rates, latency, throughput
- **Track**: GCR auto-scaling behavior
- **Note**: Any anomalies or issues

### After Test:
- **Generate**: HTML report from .jtl file
- **Capture**: Dashboard screenshots
- **Document**: Findings and recommendations
- **Share**: Results with team

---

## 📊 Success Criteria (from test_plan.md)

Your tests will be considered successful if:

### GCR Performance
- ✅ Availability >= 99.5%
- ✅ Average Latency <= 250ms
- ✅ Handles 10k TPS successfully
- ✅ No spike in 429/500/503/504 errors

### System Behavior
- ✅ PubSub recovers to 0 unacked messages
- ✅ GCR instances scale appropriately
- ✅ CPU/Memory within acceptable range
- ✅ No 401/429 errors from Kirby

---

## ❓ Need Help?

### Questions About:
- **JMeter**: Review JMETER_SCRIPT_GUIDE.md or check JMeter documentation
- **Test Plan**: Review test_plan.md or contact Platform team
- **Execution**: Review EXECUTION_GUIDE.md
- **Results**: Check Datadog dashboards or consult with D&A team

### Issues During:
- **Script Creation**: Start with sample scripts, debug with 1 thread
- **Test Execution**: Check firewall/VPN, verify authentication
- **Result Analysis**: Compare against success criteria

---

## 🎉 You're All Set!

Everything is prepared for you to create the JMeter scripts. The documentation is comprehensive and includes step-by-step instructions.

**Your main task now**: Follow the JMETER_SCRIPT_GUIDE.md to create the two .jmx files.

**Estimated Time**: 
- Reading documentation: 30 minutes
- Creating scripts: 1-2 hours
- Testing locally: 30 minutes
- **Total**: ~3 hours

Good luck! 🚀
