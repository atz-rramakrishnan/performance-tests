# CPI Performance Tests - Execution Guide

## Quick Start

### Prerequisites
- JMeter 5.6.3+ installed
- Java 8+ installed
- Access to QA environment
- VPN connection (if required)

### Running Tests Locally (Smoke Test)

```powershell
# Navigate to JMeter bin directory
cd C:\Tools\apache-jmeter-5.6.3\bin

# Run Gradual Ramp-Up Test (small scale)
.\jmeter.bat -n `
  -t "C:\path\to\CPI_Logs_Gradual_Ramp.jmx" `
  -l ".\results\gradual-test-results.jtl" `
  -e -o ".\results\gradual-html-report" `
  -JbaseUrl=api.qa.aritzia.com `
  -JtargetTPS=100 `
  -JtestDuration=60

# Run Sudden Burst Test (small scale)
.\jmeter.bat -n `
  -t "C:\path\to\CPI_Logs_Sudden_Burst.jmx" `
  -l ".\results\burst-test-results.jtl" `
  -e -o ".\results\burst-html-report" `
  -JbaseUrl=api.qa.aritzia.com `
  -JtargetTPS=100 `
  -JtestDuration=60
```

### Running Full-Scale Tests on VM

```bash
#!/bin/bash
# On Linux VM

cd /opt/jmeter/bin

# Gradual Ramp-Up Test (full scale)
./jmeter.sh -n \
  -t /path/to/CPI_Logs_Gradual_Ramp.jmx \
  -l /results/gradual-full-$(date +%Y%m%d-%H%M%S).jtl \
  -e -o /results/gradual-full-report \
  -JbaseUrl=api.qa.aritzia.com \
  -JtargetTPS=5000 \
  -JtestDuration=1800

# Sudden Burst Test (full scale)
./jmeter.sh -n \
  -t /path/to/CPI_Logs_Sudden_Burst.jmx \
  -l /results/burst-full-$(date +%Y%m%d-%H%M%S).jtl \
  -e -o /results/burst-full-report \
  -JbaseUrl=api.qa.aritzia.com \
  -JtargetTPS=5000 \
  -JtestDuration=900
```

## Test Parameters

| Parameter | Description | Default | Gradual | Burst |
|-----------|-------------|---------|---------|-------|
| `baseUrl` | API base URL | api.qa.aritzia.com | Same | Same |
| `targetTPS` | Target transactions/sec | 5000 | 5000 | 5000 |
| `testDuration` | Test duration in seconds | Varies | 1800 (30min) | 900 (15min) |

## Monitoring During Tests

### Datadog Dashboards to Watch
1. **GCR Performance**: Monitor latency, throughput, errors
2. **PubSub Metrics**: Watch queue depth and message rate
3. **Apigee Metrics**: Track API performance
4. **Kirby Metrics**: Coordinate with D&A team

### Key Metrics to Track
- Response Time (p50, p95, p99)
- Throughput (requests/second)
- Error Rate (%)
- GCR Instance Count
- CPU & Memory Utilization

## Interpreting Results

### JMeter Results (.jtl file)
Open in JMeter GUI or generate HTML report:
```powershell
jmeter -g results.jtl -o html-report-folder
```

### Success Criteria
- [ ] 99.5%+ availability
- [ ] Avg latency <= 250ms
- [ ] < 1% error rate
- [ ] No GCR 429/500/503 spikes

## Troubleshooting

### Test won't start
- Check Java installation: `java -version`
- Verify JMeter installation
- Check test file path is correct

### Connection errors
- Verify VPN connection
- Check firewall rules
- Confirm baseUrl is correct

### High error rate
- Check API authentication
- Verify payload format
- Review server logs

### Low throughput
- Increase thread count
- Check network bandwidth
- Verify Constant Throughput Timer settings

## Post-Test Actions

1. **Generate Report**:
   ```powershell
   jmeter -g results.jtl -o html-report
   ```

2. **Archive Results**:
   - Save .jtl file
   - Save HTML report
   - Take Datadog dashboard screenshots

3. **Document Findings**:
   - Any errors encountered
   - Performance bottlenecks
   - Recommendations for improvement

4. **Share Results**:
   - Email report to team
   - Update Confluence page
   - Discuss in team meeting

## Contact

For questions or issues:
- Platform Team: [team email]
- D&A Team (Kirby): [team email]
