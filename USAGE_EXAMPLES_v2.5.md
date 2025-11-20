# Usage Examples: v2.5

## Quick Start

### Basic Usage
```bash
python nxos_compliance_checker_v2.5.py config.txt
```

### With Custom Policy
```bash
python nxos_compliance_checker_v2.5.py config.txt --policy custom_policy.yaml
```

### Verbose Mode
```bash
python nxos_compliance_checker_v2.5.py config.txt --verbose
```

---

## Output Formats

### Console Output (Default)
```bash
python nxos_compliance_checker_v2.5.py config.txt
```

**Sample Output:**
```
================================================================================
Production Compliance Checker v2.5 (Unified Value Checking)
================================================================================
Device:     ARAAUS32
Config:     config.txt
Policy:     Cisco NX-OS Security Hardening - v2.5 v2.5
Scan Time:  2025-11-19 15:30:45
================================================================================

Running System Hardening checks (4 checks)...
Running Management Plane checks (49 checks)...
Running Control Plane checks (21 checks)...

================================================================================
❌ FAILED CHECKS (Require Immediate Attention)
================================================================================

1. ❌ [FAIL] [NX-SYS-002] Bash Shell Consent Token Required
   Category:   System Hardening
   Severity:   HIGH
   Status:     FAIL
   Message:    ❌ Required configuration not found: terminal password.*consent-token

================================================================================
📊 COMPLIANCE SUMMARY
================================================================================

Overall Score: 456/520 (87.7%)
Final Grade:   B (✅ GOOD)

Status                Count      Percentage
--------------------------------------------------
✅ [PASS] Passed       85         81.7%
❌ [FAIL] Failed       12         11.5%
⚠️  [WARN] Warnings     7          6.7%
--------------------------------------------------
Total                 104         100.0%
```

### JSON Output
```bash
python nxos_compliance_checker_v2.5.py config.txt \
    --format json --output results.json
```

**Sample Output (results.json):**
```json
{
  "metadata": {
    "device": "ARAAUS32",
    "config_file": "config.txt",
    "policy": {
      "policy_name": "Cisco NX-OS Security Hardening - v2.5",
      "version": "2.4-clean"
    },
    "timestamp": "2025-11-19T15:30:45.123456",
    "checker_version": "2.4-clean"
  },
  "summary": {
    "score": 456,
    "max_score": 520,
    "percentage": 87.7,
    "grade": "B",
    "passed": 85,
    "failed": 12,
    "warnings": 7,
    "errors": 0,
    "total": 104
  },
  "results": [
    {
      "id": "NX-MP-005AB",
      "name": "TACACS+ Timeout Absolute Value (30 seconds)",
      "category": "AAA Configuration",
      "severity": "MEDIUM",
      "status": "PASS",
      "message": "✅ Value matches required: 30 seconds",
      "score": 5,
      "max_score": 5
    }
  ]
}
```

### CSV Output
```bash
python nxos_compliance_checker_v2.5.py config.txt \
    --format csv --output results.csv
```

**Sample Output (results.csv):**
```csv
id,name,category,severity,status,message,score,max_score,remediation,reference
NX-MP-005AA,TACACS+ Server Timeout Configured,AAA Configuration,CRITICAL,PASS,✅ Found: tacacs-server timeout 30,10,10,...,...
NX-MP-005AB,TACACS+ Timeout Absolute Value,AAA Configuration,MEDIUM,PASS,✅ Value matches required: 30 seconds,5,5,...,...
```

### Syslog Format
```bash
python nxos_compliance_checker_v2.5.py config.txt \
    --format syslog --output compliance.log
```

**Sample Output (compliance.log):**
```
timestamp=2025-11-19 15:30:45 hostname=ARAAUS32 check_id=NX-MP-005AA check_name="TACACS+ Server Timeout Configured" category="AAA Configuration" severity=CRITICAL status=PASS score=10 max_score=10 message="✅ Found: tacacs-server timeout 30"
timestamp=2025-11-19 15:30:45 hostname=ARAAUS32 check_id=NX-MP-005AB check_name="TACACS+ Timeout Absolute Value" category="AAA Configuration" severity=MEDIUM status=PASS score=5 max_score=5 message="✅ Value matches required: 30 seconds"
```

---

## Policy File Examples

### Example 1: Absolute Value Check (Mandatory)

```yaml
- id: NX-MP-005AB
  name: TACACS+ Timeout Absolute Value (30 seconds)
  description: Verify TACACS+ timeout is configured to exactly 30 seconds
  category: AAA Configuration
  severity: MEDIUM
  check_type: value
  pattern: tacacs-server timeout
  comparison: equals
  expected_value: 30
  unit: ' seconds'
  remediation: 'Configure: tacacs-server timeout 30'
  reference: Management Plane > AAA
  score: 5
```

**Configuration:**
```
tacacs-server timeout 30    → ✅ PASS
tacacs-server timeout 25    → ❌ FAIL
tacacs-server timeout 35    → ❌ FAIL
(not configured)            → ❌ FAIL
```

### Example 2: Range Check (Advisory)

```yaml
- id: NX-MP-006AB
  name: RADIUS Timeout Optimal Range
  description: Verify RADIUS timeout is within 3-10 second optimal range
  category: AAA Configuration
  severity: MEDIUM
  check_type: value
  pattern: radius-server timeout
  comparison: range
  min_value: 3
  max_value: 10
  unit: ' seconds'
  remediation: 'Adjust to optimal range: radius-server timeout 5'
  reference: Management Plane > AAA
  score: 2
```

**Configuration:**
```
radius-server timeout 5     → ✅ PASS (optimal)
radius-server timeout 3     → ✅ PASS (min edge)
radius-server timeout 10    → ✅ PASS (max edge)
radius-server timeout 2     → ⚠️  WARN (below range)
radius-server timeout 15    → ⚠️  WARN (above range)
(not configured)            → ❌ FAIL
```

### Example 3: Backward Compatible (Old Format)

```yaml
- id: NX-MP-006AB
  name: RADIUS Timeout Optimal Range
  check_type: value_range  # ← OLD FORMAT (still works!)
  pattern: radius-server timeout
  min_value: 3
  max_value: 10
  severity: MEDIUM
  score: 2
```

**Result:** Automatically treated as `comparison: range`

### Example 4: Greater Than Check

```yaml
- id: NX-MP-032
  name: SSH Session Limit
  description: Verify SSH session limit is greater than 5
  category: Secure Management
  severity: LOW
  check_type: value
  pattern: ssh session-limit
  comparison: greater_than
  expected_value: 5
  unit: ' sessions'
  remediation: 'Configure: ssh session-limit 10'
  reference: Management Plane > SSH
  score: 2
```

**Configuration:**
```
ssh session-limit 10        → ✅ PASS (10 > 5)
ssh session-limit 6         → ✅ PASS (6 > 5)
ssh session-limit 5         → ❌ FAIL (5 not > 5)
ssh session-limit 3         → ❌ FAIL (3 not > 5)
```

### Example 5: Less Than Check

```yaml
- id: NX-MP-033
  name: STP Hello Time
  description: Verify STP hello time is less than 10 seconds
  category: Data Plane
  severity: LOW
  check_type: value
  pattern: spanning-tree hello-time
  comparison: less_than
  expected_value: 10
  unit: ' seconds'
  remediation: 'Configure: spanning-tree hello-time 5'
  reference: Data Plane > STP
  score: 2
```

**Configuration:**
```
spanning-tree hello-time 5  → ✅ PASS (5 < 10)
spanning-tree hello-time 9  → ✅ PASS (9 < 10)
spanning-tree hello-time 10 → ❌ FAIL (10 not < 10)
spanning-tree hello-time 15 → ❌ FAIL (15 not < 10)
```

---

## Real-World Scenarios

### Scenario 1: Test Your Current Config

```bash
# Step 1: Run compliance check
python nxos_compliance_checker_v2.5.py config.txt --verbose

# Step 2: Review results
cat config_compliance_*.json | jq '.summary'

# Step 3: Fix critical issues
# (Address FAIL checks first)

# Step 4: Re-run check
python nxos_compliance_checker_v2.5.py config.txt --verbose
```

### Scenario 2: Compare Multiple Devices

```bash
# Check device 1
python nxos_compliance_checker_v2.5.py device1.txt \
    --format json --output device1_results.json

# Check device 2
python nxos_compliance_checker_v2.5.py device2.txt \
    --format json --output device2_results.json

# Compare scores
jq '.summary.percentage' device1_results.json
jq '.summary.percentage' device2_results.json
```

### Scenario 3: CI/CD Pipeline Integration

```bash
#!/bin/bash
# compliance_check.sh

CONFIG_FILE=$1
THRESHOLD=80

# Run compliance check
python nxos_compliance_checker_v2.5.py $CONFIG_FILE \
    --format json --output results.json

# Extract score
SCORE=$(jq '.summary.percentage' results.json)

# Check threshold
if (( $(echo "$SCORE < $THRESHOLD" | bc -l) )); then
    echo "❌ Compliance score $SCORE% below threshold $THRESHOLD%"
    exit 1
else
    echo "✅ Compliance score $SCORE% meets threshold $THRESHOLD%"
    exit 0
fi
```

### Scenario 4: Splunk Integration

```bash
# Generate syslog format
python nxos_compliance_checker_v2.5.py config.txt \
    --format syslog --output compliance.log

# Send to Splunk HEC
curl -k https://splunk.example.com:8088/services/collector/raw \
    -H "Authorization: Splunk YOUR-HEC-TOKEN" \
    -d "$(cat compliance.log)"
```

### Scenario 5: Scheduled Compliance Monitoring

```bash
# crontab entry - run daily at 2 AM
0 2 * * * /usr/local/bin/compliance_check.sh >> /var/log/compliance.log 2>&1
```

**compliance_check.sh:**
```bash
#!/bin/bash
DATE=$(date +%Y%m%d)
DEVICE_LIST="router1.txt router2.txt router3.txt"

for CONFIG in $DEVICE_LIST; do
    python nxos_compliance_checker_v2.5.py $CONFIG \
        --format json --output "${CONFIG%.txt}_${DATE}.json"
done

# Archive results
tar -czf compliance_${DATE}.tar.gz *_${DATE}.json
mv compliance_${DATE}.tar.gz /archive/
```

---

## Troubleshooting Examples

### Example 1: Debug Value Check

```bash
# Run with verbose mode to see detailed processing
python nxos_compliance_checker_v2.5.py config.txt --verbose 2>&1 | grep "NX-MP-005AB"
```

**Output:**
```
  ✅ [PASS] [NX-MP-005AB] TACACS+ Timeout Absolute Value (30 seconds): PASS
```

### Example 2: Check Specific Category

```bash
# Use grep to filter results
python nxos_compliance_checker_v2.5.py config.txt --verbose 2>&1 | grep "AAA Configuration"
```

### Example 3: Export Failed Checks Only

```bash
# Run check and save to JSON
python nxos_compliance_checker_v2.5.py config.txt \
    --format json --output results.json

# Extract failed checks
jq '.results[] | select(.status == "FAIL")' results.json
```

### Example 4: Generate Remediation Report

```bash
# Extract failed checks with remediation
jq '.results[] | select(.status == "FAIL") | {id, name, remediation}' results.json
```

---

## Performance Benchmarks

### Small Config (100 lines)
```
Time: ~0.5 seconds
Checks: 104
Memory: ~20 MB
```

### Medium Config (1,000 lines)
```
Time: ~1.2 seconds
Checks: 104
Memory: ~25 MB
```

### Large Config (10,000 lines)
```
Time: ~3.5 seconds
Checks: 104
Memory: ~40 MB
```

---

## Best Practices

### 1. Always Use Verbose Mode for Debugging
```bash
python nxos_compliance_checker_v2.5.py config.txt --verbose
```

### 2. Save Results for Historical Tracking
```bash
DATE=$(date +%Y%m%d)
python nxos_compliance_checker_v2.5.py config.txt \
    --format json --output "compliance_${DATE}.json"
```

### 3. Test New Policies in Non-Production
```bash
# Test policy changes on backup config first
python nxos_compliance_checker_v2.5.py backup_config.txt \
    --policy new_policy.yaml --verbose
```

### 4. Use JSON Format for Automation
```bash
# JSON is easier to parse in scripts
python nxos_compliance_checker_v2.5.py config.txt \
    --format json --output results.json
```

### 5. Monitor Compliance Trends
```bash
# Track scores over time
jq '.summary.percentage' compliance_*.json | \
    awk '{sum+=$1; count++} END {print "Average:", sum/count}'
```

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────────┐
│           v2.5 Quick Reference                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Basic Check:                                               │
│    python nxos_compliance_checker_v2.5.py \       │
│        config.txt                                           │
│                                                             │
│  With Custom Policy:                                        │
│    python nxos_compliance_checker_v2.5.py \       │
│        config.txt --policy custom.yaml                      │
│                                                             │
│  JSON Output:                                               │
│    --format json --output results.json                      │
│                                                             │
│  CSV Output:                                                │
│    --format csv --output results.csv                        │
│                                                             │
│  Syslog Output:                                             │
│    --format syslog --output compliance.log                  │
│                                                             │
│  Verbose Mode:                                              │
│    --verbose                                                │
│                                                             │
│  Exit Codes:                                                │
│    0 = All checks passed                                    │
│    1 = Some checks failed/errors                            │
│    2 = Fatal error                                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Support

For issues or questions:
- Review `MIGRATION_GUIDE_v2.5.md`
- Check `CHANGELOG_v2.5.md`
- Use `--verbose` mode for debugging
- Review policy file syntax
