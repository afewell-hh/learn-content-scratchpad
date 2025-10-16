# Module 3.2 Design: Dashboard Interpretation

## Module Metadata

- **Module Number:** 3.2
- **Module Title:** Dashboard Interpretation
- **Course:** Course 3 - Observability & Fabric Health
- **Estimated Duration:** 14-16 minutes
  - Introduction: 2 minutes
  - Core Concepts: 6-7 minutes
  - Hands-On Lab: 5-6 minutes
  - Wrap-Up & Assessment: 2 minutes

- **Version:** 1.0
- **Date:** 2025-10-16
- **Status:** DESIGN - Ready for Review

- **Prerequisites:**
  - ✅ Module 3.1 Complete (Fabric Telemetry Overview)
  - ✅ Understanding of Prometheus metrics and PromQL
  - ✅ Familiarity with metric types (counters, gauges)
  - ✅ Basic Grafana navigation (from Module 1.3)

## Learning Objectives

By the end of this module, learners will be able to:

1. **Interpret Fabric Dashboard** - Read VPC count, VNI allocation, and BGP session status
2. **Interpret Interfaces Dashboard** - Understand interface state, VLAN config, traffic rates, and errors
3. **Interpret Platform Dashboard** - Read switch resource usage (CPU, memory, disk, temperature)
4. **Interpret Logs Dashboard** - Filter and search switch logs for errors and events
5. **Interpret Node Exporter Dashboard** - Understand detailed Linux system metrics
6. **Interpret Switch Critical Resources Dashboard** - Identify ASIC resource exhaustion risks
7. **Create morning health check workflow** - Use dashboards systematically for daily monitoring

**Bloom's Taxonomy Level**: Apply, Analyze (interpreting data, identifying patterns, making decisions)

## Content Outline

### Introduction (2 minutes)

**Hook: From Raw Metrics to Actionable Insights**

> In Module 3.1, you learned how telemetry flows from switches to Prometheus and ran PromQL queries to see raw metrics. You queried CPU percentages, bandwidth rates, and BGP neighbor states.
>
> But in daily operations, you won't be writing PromQL queries every time you want to check fabric health. That's where **Grafana dashboards** come in—pre-built visualizations that answer common operational questions at a glance.

**The Morning Question:**

Every morning, a fabric operator asks:
- **Is my fabric healthy?**
- **Are all BGP sessions up?**
- **Are there any interface errors?**
- **Is any switch running hot or out of resources?**

Grafana dashboards answer these questions in seconds without writing queries.

**What You'll Learn:**

Hedgehog provides **6 pre-built Grafana dashboards**:
1. **Fabric Dashboard** - BGP underlay health
2. **Interfaces Dashboard** - Interface state and traffic
3. **Platform Dashboard** - Hardware health (PSU, fans, temperature)
4. **Logs Dashboard** - Switch logs and error filtering
5. **Node Exporter Dashboard** - Linux system metrics
6. **Switch Critical Resources Dashboard** - ASIC resource limits

You'll learn to **read** each dashboard (not build them), identify healthy vs unhealthy states, and create a morning health check workflow.

**Context: Proactive vs Reactive Monitoring**

- **Reactive**: Wait for alerts or user reports, then investigate
- **Proactive**: Check dashboards daily, spot trends before they become problems

This module teaches proactive monitoring—catching issues early.

---

### Core Concepts (6-7 minutes)

#### Concept 1: Fabric Dashboard - BGP Underlay Health

**Purpose:** Monitor the BGP fabric underlay that connects all switches

**Key Panels:**

**1. BGP Sessions Overview**
- **Metric:** Count of BGP sessions (total, established, down)
- **Healthy State:** All sessions = Established
- **Unhealthy State:** Any sessions in Idle, Connect, or Active state
- **Example:**
  ```
  Total Sessions: 40
  Established: 40
  Down: 0
  ```

**2. BGP Neighbor Status Table**
- **Columns:** Switch, Neighbor IP, State, Prefixes Received, Uptime
- **Healthy Row:** State = "established", Prefixes > 0, Uptime > 1 hour
- **Unhealthy Row:** State ≠ "established", Prefixes = 0
- **Example:**
  ```
  Switch    Neighbor         State         Prefixes  Uptime
  leaf-01   172.30.128.1    established   12        3d 5h
  leaf-01   172.30.128.2    established   12        3d 5h
  leaf-02   172.30.128.1    Idle          0         0s      ← PROBLEM
  ```

**3. BGP Session Flaps**
- **Metric:** BGP session state changes over time
- **Healthy State:** Flat line (no changes)
- **Unhealthy State:** Spikes indicate session instability
- **Example:** Graph showing session going down and back up = flapping

**4. Prefixes Received/Advertised**
- **Metric:** Number of routes exchanged per neighbor
- **Healthy State:** Consistent count (e.g., 10-20 prefixes per neighbor)
- **Unhealthy State:** Sudden drop to 0 (neighbor lost routes)

**Dashboard Actions:**

- **Daily Health Check:** Verify BGP Sessions count matches expected (e.g., 40)
- **Troubleshooting:** Identify which switch/neighbor has session down
- **Trend Analysis:** Check for flapping patterns over last 24 hours

**What "Healthy" Looks Like:**
- All sessions green/established
- Stable prefix counts
- No flaps in last 24 hours
- Uptime > last change window

---

#### Concept 2: Interfaces Dashboard - Traffic and Errors

**Purpose:** Monitor interface state, traffic rates, and error counters

**Key Panels:**

**1. Interface Operational State**
- **Metric:** Up (green) or Down (red) per interface
- **Healthy State:** All configured interfaces up
- **Unhealthy State:** Expected-up interfaces showing down
- **Example:**
  ```
  leaf-01/Ethernet1: UP    (server connection)
  leaf-01/Ethernet2: UP    (server connection)
  leaf-01/Ethernet48: UP   (spine uplink)
  leaf-01/Ethernet49: DOWN (unused port - OK)
  ```

**2. Interface Traffic Rate**
- **Metric:** Bits per second (bps) or packets per second (pps)
- **Visualization:** Time series graph per interface
- **Healthy State:** Consistent traffic matching workload
- **Unhealthy State:** Unexpected spikes or drops to zero
- **Example:**
  - Server interface: 2 Gbps steady (expected)
  - Server interface: 10 Gbps sustained (possible saturation - investigate)

**3. Interface Utilization**
- **Metric:** Percentage of link capacity used
- **Healthy State:** < 70% utilization (headroom available)
- **Warning State:** 70-90% (nearing capacity)
- **Critical State:** > 90% (congestion risk)
- **Example:**
  ```
  Ethernet1: 45% (healthy)
  Ethernet2: 85% (warning - consider upgrade)
  Ethernet3: 95% (critical - likely packet drops)
  ```

**4. Error Counters**
- **Metric:** CRC errors, frame errors, discards per interface
- **Healthy State:** 0 errors, or very low error rate
- **Unhealthy State:** Growing error count (cable issue, duplex mismatch)
- **Example:**
  ```
  Interface    CRC Errors   Frame Errors   Discards
  Ethernet1    0            0              0          ← Healthy
  Ethernet2    1,234        523            0          ← Cable problem
  Ethernet3    0            0              52,341     ← Congestion (drops)
  ```

**5. VLAN Configuration**
- **Metric:** VLANs configured on each interface
- **Use Case:** Verify VPCAttachment applied correct VLANs
- **Example:**
  ```
  Ethernet1: VLAN 1010, 1020 (expected for server-01)
  Ethernet2: VLAN 1030        (expected for server-02)
  ```

**Dashboard Actions:**

- **Daily Health Check:** Scan for red (down) interfaces or high error rates
- **Capacity Planning:** Identify interfaces consistently > 70% utilized
- **Troubleshooting:** Correlate traffic drop with VPC connectivity issues
- **Validation:** After VPCAttachment, verify VLAN appears on interface

**What "Healthy" Looks Like:**
- All expected interfaces up (green)
- Error counters flat (not increasing)
- Utilization < 70%
- Traffic patterns match workload expectations

---

#### Concept 3: Platform Dashboard - Hardware Health

**Purpose:** Monitor switch hardware (PSU, fans, temperature, optics)

**Key Panels:**

**1. PSU Status**
- **Metric:** Operational state (OK/Failed) per PSU
- **Healthy State:** All PSUs = OK
- **Unhealthy State:** Any PSU = Failed or Not Present
- **Example:**
  ```
  Switch    PSU1   PSU2
  leaf-01   OK     OK       ← Healthy
  leaf-02   OK     FAILED   ← Replace PSU2
  ```

**2. PSU Voltage**
- **Metric:** Input/output voltage in volts
- **Healthy State:** Within expected range (e.g., 11.5V - 12.5V for 12V rail)
- **Unhealthy State:** Voltage outside range (power supply issue)
- **Example:**
  ```
  PSU1 Output: 12.1V (OK)
  PSU2 Output: 10.2V (Low - failing PSU)
  ```

**3. Fan Speeds**
- **Metric:** RPM (revolutions per minute) per fan
- **Healthy State:** All fans > minimum threshold (e.g., > 3000 RPM)
- **Unhealthy State:** Fan = 0 RPM (failed) or very low
- **Example:**
  ```
  Fan Tray 1: 5,200 RPM (OK)
  Fan Tray 2: 5,100 RPM (OK)
  Fan Tray 3: 0 RPM     (FAILED - thermal risk)
  ```

**4. Temperature Sensors**
- **Metric:** Celsius per sensor (CPU, ASIC, ambient, PSU)
- **Healthy State:** Below warning thresholds
  - CPU: < 70°C
  - ASIC: < 80°C
  - Ambient: < 45°C
- **Unhealthy State:** Approaching or exceeding limits
- **Example:**
  ```
  CPU Temp: 55°C (OK)
  ASIC Temp: 85°C (WARNING - check cooling)
  Ambient: 48°C (WARNING - data center HVAC issue)
  ```

**5. Transceiver Optics (DOM)**
- **Metric:** TX/RX power, temperature per optical interface
- **Healthy State:** Within optic specifications
- **Unhealthy State:** Low RX power (dirty/bad fiber), high temp (failing optic)
- **Example:**
  ```
  Ethernet48 (Spine Uplink):
    TX Power: -2.5 dBm (OK)
    RX Power: -3.1 dBm (OK)
    Temp: 45°C (OK)

  Ethernet49:
    RX Power: -15.2 dBm (CRITICAL - signal loss, check fiber)
  ```

**Dashboard Actions:**

- **Daily Health Check:** Scan for failed PSUs, stopped fans, high temps
- **Preventive Maintenance:** Schedule PSU/fan replacement before failure
- **Capacity Planning:** Track temperature trends (data center cooling)
- **Optics Validation:** After fiber installation, verify RX power in range

**What "Healthy" Looks Like:**
- All PSUs operational
- All fans running > minimum RPM
- Temperatures well below limits
- Optic power levels within spec

---

#### Concept 4: Logs Dashboard - Error Filtering

**Purpose:** Aggregate and search switch logs for troubleshooting

**Key Panels:**

**1. Syslog Stream**
- **Content:** Live stream of syslog messages from all switches
- **Use Case:** Watch log messages in real-time
- **Filter By:** Log level, switch, time range
- **Example:**
  ```
  [leaf-01] Oct 16 10:15:23 INFO: BGP neighbor 172.30.128.1 established
  [leaf-02] Oct 16 10:15:45 ERROR: Interface Ethernet5 link down
  [spine-01] Oct 16 10:16:02 WARNING: High CPU usage detected
  ```

**2. Log Level Breakdown**
- **Metric:** Count of logs by severity (ERROR, WARNING, INFO, DEBUG)
- **Healthy State:** Few or no ERRORs
- **Unhealthy State:** Spike in ERROR count
- **Example:**
  ```
  Last 1 hour:
    ERROR: 2
    WARNING: 12
    INFO: 523
  ```

**3. Error Rate Visualization**
- **Metric:** Errors per minute over time
- **Healthy State:** Flat line near zero
- **Unhealthy State:** Spike indicates incident
- **Example:** Graph shows spike at 10:15 (correlate with interface down event)

**4. Log Search / Filtering**
- **Capability:** Full-text search across all logs
- **Use Cases:**
  - Find all BGP-related errors: `bgp AND error`
  - Find logs for specific switch: `switch="leaf-01"`
  - Find interface events: `interface AND (up OR down)`

**Dashboard Actions:**

- **Daily Health Check:** Check ERROR count (should be 0 or near 0)
- **Troubleshooting:** Search for specific error messages
- **Incident Investigation:** Correlate log spike with metrics change
- **Audit Trail:** Review configuration change logs

**What "Healthy" Looks Like:**
- ERROR count = 0 or very low
- No unusual log patterns
- Warnings are expected/known (e.g., unused ports)

---

#### Concept 5: Node Exporter Dashboard - System Metrics

**Purpose:** Deep dive into Linux OS metrics (CPU, memory, disk, network I/O)

**Key Panels:**

**1. CPU Utilization**
- **Metric:** Percentage breakdown (user, system, idle, iowait)
- **Healthy State:** < 70% total utilization, low iowait
- **Unhealthy State:** > 80% sustained, high iowait (disk bottleneck)
- **Example:**
  ```
  User: 25%
  System: 15%
  IOWait: 5%
  Idle: 55%
  Total: 45% (healthy)
  ```

**2. Load Average**
- **Metric:** 1-min, 5-min, 15-min load average
- **Healthy State:** Load < number of CPUs
- **Unhealthy State:** Load > CPUs (queued processes)
- **Example:**
  ```
  1-min: 2.5
  5-min: 2.2
  15-min: 1.8
  (Assuming 4 CPUs: healthy - load < 4)
  ```

**3. Memory Usage**
- **Metric:** Total, used, available, buffers/cache
- **Healthy State:** Available > 20% of total
- **Unhealthy State:** Available < 10% (memory pressure)
- **Example:**
  ```
  Total: 16 GB
  Used: 10 GB
  Available: 5 GB (31% - healthy)
  ```

**4. Disk Space**
- **Metric:** Used/available per filesystem
- **Healthy State:** < 80% used
- **Unhealthy State:** > 90% used (risk of full disk)
- **Example:**
  ```
  /: 45% used (OK)
  /var/log: 92% used (WARNING - rotate logs)
  ```

**5. Disk I/O**
- **Metric:** Read/write operations per second, throughput
- **Use Case:** Identify disk bottlenecks
- **Example:**
  ```
  Read: 150 IOPS, 25 MB/s
  Write: 300 IOPS, 50 MB/s
  ```

**6. Network Throughput (All Interfaces)**
- **Metric:** Total bytes in/out across all network interfaces
- **Use Case:** Overall switch traffic
- **Example:**
  ```
  In: 5 Gbps
  Out: 4.8 Gbps
  ```

**Dashboard Actions:**

- **Performance Troubleshooting:** Identify if switch CPU/memory is bottleneck
- **Capacity Planning:** Track resource usage trends over time
- **Disk Management:** Proactively clear logs before disk fills
- **Baseline Understanding:** Know "normal" resource usage for comparison

**What "Healthy" Looks Like:**
- CPU < 70%, low iowait
- Memory available > 20%
- Disk usage < 80%
- Load average < CPU count

---

#### Concept 6: Switch Critical Resources Dashboard - ASIC Limits

**Purpose:** Monitor programmable ASIC hardware table capacity (prevents resource exhaustion)

**Key Panels:**

**1. IPv4 Route Table**
- **Metric:** Routes used / routes available
- **Healthy State:** < 80% capacity
- **Critical State:** > 90% (risk of route installation failure)
- **Example:**
  ```
  leaf-01: 15,000 / 32,768 routes (46% - OK)
  spine-01: 28,000 / 32,768 routes (85% - WARNING)
  ```

**2. IPv4 Nexthop Table**
- **Metric:** Nexthops used / available
- **Use Case:** Tracks ECMP paths
- **Example:**
  ```
  Used: 2,048 / 4,096 (50% - OK)
  ```

**3. IPv4 Neighbor (ARP) Table**
- **Metric:** ARP entries used / available
- **Healthy State:** < 80% capacity
- **Risk:** Large subnets with many hosts can exhaust ARP table
- **Example:**
  ```
  Used: 8,192 / 16,384 (50% - OK)
  Used: 15,500 / 16,384 (95% - CRITICAL)
  ```

**4. FDB (Forwarding Database) Capacity**
- **Metric:** MAC addresses learned / capacity
- **Use Case:** Layer 2 forwarding table
- **Example:**
  ```
  Used: 12,000 / 32,768 MACs (37% - OK)
  ```

**5. ACL Table Usage**
- **Metric:** ACL entries used / available
- **Use Case:** Security rules, permit lists
- **Example:**
  ```
  Used: 512 / 2,048 (25% - OK)
  ```

**6. IPMC (IP Multicast) Table**
- **Metric:** Multicast entries used / available
- **Use Case:** Multicast routing (if enabled)

**Dashboard Actions:**

- **Daily Health Check:** Verify no resource > 90% used
- **Capacity Planning:** Identify resources trending toward limits
- **Scale Planning:** If ARP table nearing limit, consider subnet redesign
- **Alert Thresholds:** Set alerts at 80% capacity

**What "Healthy" Looks Like:**
- All resources < 80% capacity
- Stable usage (not rapidly growing)
- Headroom for growth

**What "Unhealthy" Looks Like:**
- Any resource > 90% (immediate risk)
- Rapidly growing usage (trend toward exhaustion)

---

### Hands-On Lab (5-6 minutes)

**Lab Title:** Morning Health Check Workflow Using Grafana Dashboards

**Scenario:**

It's Monday morning, and you're the on-call fabric operator. Your first task: verify the fabric is healthy before the business day begins. You'll use Grafana dashboards to perform a systematic health check.

**Environment Access:**
- **Grafana:** http://localhost:3000 (username: `admin`, password: `prom-operator`)

---

#### Task 1: Check BGP Fabric Health (2 minutes)

**Objective:** Verify all BGP sessions are established

**Steps:**

1. **Open Grafana:**
   - Navigate to http://localhost:3000
   - Sign in (admin / prom-operator)

2. **Access Fabric Dashboard:**
   - Click **Dashboards** (left sidebar)
   - Select **"Hedgehog Fabric"** dashboard

3. **Check BGP Sessions Overview Panel:**
   - Locate "BGP Sessions" panel (top of dashboard)
   - **Expected healthy state:**
     ```
     Total Sessions: 40
     Established: 40
     Down: 0
     ```
   - ✅ If Established = Total, fabric underlay is healthy
   - ❌ If Down > 0, identify which neighbor is down

4. **Review BGP Neighbor Table:**
   - Scroll to "BGP Neighbor Status" table
   - Scan "State" column - all should say "established"
   - Check "Prefixes Received" - should be > 0
   - **Red flag:** Any row with State ≠ "established"

5. **Check for Session Flaps:**
   - Find "BGP Session Flaps" time series panel
   - Set time range to "Last 24 hours"
   - **Healthy:** Flat line (no state changes)
   - **Unhealthy:** Spikes indicate sessions going up/down

**Success Criteria:**
- ✅ All BGP sessions established
- ✅ No flaps in last 24 hours
- ✅ Prefix counts stable

**If Unhealthy:**
- Note which switch and neighbor has issue
- Proceed to troubleshooting in Module 3.3 (Events & Logs correlation)

---

#### Task 2: Check Interface Health (1-2 minutes)

**Objective:** Verify all expected interfaces are up and error-free

**Steps:**

1. **Access Interfaces Dashboard:**
   - Click **Dashboards** → **"Hedgehog Interfaces"**

2. **Check Interface State Overview:**
   - Locate "Interface Operational State" panel
   - **Healthy:** Expected interfaces show green (Up)
   - **Unhealthy:** Expected interfaces show red (Down)
   - **Note:** Unused ports down = OK

3. **Check Error Counters:**
   - Find "Interface Errors" panel or table
   - **Healthy:** Error rate = 0 or flat line
   - **Unhealthy:** Growing error count (indicates problem)
   - **Example of problem:**
     ```
     Ethernet5: CRC Errors increasing (cable issue)
     ```

4. **Check Interface Utilization:**
   - Locate "Interface Utilization %" panel
   - **Healthy:** < 70%
   - **Warning:** 70-90%
   - **Critical:** > 90%
   - **Action:** If > 90%, note for capacity planning

**Success Criteria:**
- ✅ All expected interfaces up
- ✅ No growing error counters
- ✅ Utilization < 90%

---

#### Task 3: Check Hardware Health (1 minute)

**Objective:** Verify switches are not experiencing hardware issues

**Steps:**

1. **Access Platform Dashboard:**
   - Click **Dashboards** → **"Hedgehog Platform"**

2. **Check PSU Status:**
   - Locate "PSU Status" panel
   - **Healthy:** All PSUs = OK
   - **Unhealthy:** Any PSU = Failed → Schedule replacement

3. **Check Fan Speeds:**
   - Find "Fan Speeds" panel
   - **Healthy:** All fans > 0 RPM (e.g., > 3000 RPM)
   - **Unhealthy:** Any fan = 0 RPM → Thermal risk

4. **Check Temperature:**
   - Locate "Temperature Sensors" panel
   - **Healthy thresholds:**
     - CPU < 70°C
     - ASIC < 80°C
     - Ambient < 45°C
   - **Unhealthy:** Any sensor exceeding threshold → Investigate cooling

**Success Criteria:**
- ✅ All PSUs operational
- ✅ All fans running
- ✅ Temperatures within limits

---

#### Task 4: Check for Recent Errors (1 minute)

**Objective:** Identify any error logs in last 1 hour

**Steps:**

1. **Access Logs Dashboard:**
   - Click **Dashboards** → **"Hedgehog Logs"**

2. **Check Error Count:**
   - Locate "Log Level Breakdown" panel
   - Set time range: "Last 1 hour"
   - **Healthy:** ERROR count = 0 or very low (< 5)
   - **Unhealthy:** ERROR count > 10 → Investigate

3. **Review Error Logs:**
   - If errors present, locate "Syslog Stream" panel
   - Filter by `level="error"`
   - Read error messages to identify issues
   - **Example:**
     ```
     [leaf-02] ERROR: Interface Ethernet5 link down
     ```

**Success Criteria:**
- ✅ ERROR count near zero
- ✅ No unexpected critical errors

---

#### Task 5: Optional - Check ASIC Resources (1 minute)

**Objective:** Verify no ASIC resources nearing capacity

**Steps:**

1. **Access Switch Critical Resources Dashboard:**
   - Click **Dashboards** → **"Switch Critical Resources"**

2. **Scan Resource Utilization:**
   - Review all panels (Route Table, ARP Table, FDB, ACL)
   - **Healthy:** All < 80% capacity
   - **Warning:** Any resource 80-90%
   - **Critical:** Any resource > 90%

3. **Note Resources for Capacity Planning:**
   - If any resource > 70%, document for future planning

**Success Criteria:**
- ✅ All ASIC resources < 90% capacity

---

### Wrap-Up & Assessment (2 minutes)

**What You Accomplished:**

You performed a complete fabric health check using 6 Grafana dashboards:
- ✅ Verified BGP underlay health (Fabric Dashboard)
- ✅ Checked interface state and errors (Interfaces Dashboard)
- ✅ Confirmed hardware health (Platform Dashboard)
- ✅ Scanned for recent error logs (Logs Dashboard)
- ✅ Reviewed ASIC resource usage (Critical Resources Dashboard)
- ✅ (Bonus) Node Exporter for system metrics

**Key Takeaways:**

1. **Morning health check takes < 5 minutes** with dashboards
2. **Each dashboard answers specific questions** (BGP, interfaces, hardware, logs)
3. **"Healthy" has clear criteria** (sessions up, errors = 0, resources < 80%)
4. **Dashboards enable proactive monitoring** (catch issues before alerts)
5. **Correlation across dashboards** (logs explain metrics anomalies)

**Morning Health Check Checklist:**

```
Daily Fabric Health Check (5 minutes)
────────────────────────────────────
☐ BGP sessions all established (Fabric Dashboard)
☐ No BGP flaps in last 24h
☐ Interfaces up and error-free (Interfaces Dashboard)
☐ PSUs operational, fans running (Platform Dashboard)
☐ Temperatures within limits
☐ ERROR log count near zero (Logs Dashboard)
☐ ASIC resources < 90% (Critical Resources Dashboard)
```

**Next Module Preview:**

Module 3.3: Events & Status Monitoring - You'll learn to correlate Grafana metrics with kubectl events and CRD status for complete troubleshooting.

---

### Assessment Questions

#### Question 1: BGP Health Interpretation

**Scenario:** You open the Fabric Dashboard and see:
```
Total Sessions: 40
Established: 38
Down: 2
```

What should you do next?

- A) Ignore it - 38/40 is good enough
- B) Check the BGP Neighbor Status table to identify which 2 neighbors are down
- C) Restart all switches to fix the issue
- D) Wait 1 hour and check again

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) Check the BGP Neighbor Status table to identify which 2 neighbors are down

**Explanation:**

2 down BGP sessions indicate a fabric underlay problem that needs investigation (not normal).

**Next steps:**
1. Scroll to "BGP Neighbor Status" table on Fabric Dashboard
2. Identify which switch and neighbor IPs have State ≠ "established"
3. Check kubectl events for those switches: `kubectl get events -n fab --field-selector involvedObject.name=<switch>`
4. Review Logs Dashboard for BGP-related errors on affected switches
5. If sessions remain down, escalate to support with diagnostics

**Why other options are wrong:**
- **A**: Down sessions affect fabric connectivity - not acceptable
- **C**: Restarting switches is disruptive and not appropriate first step
- **D**: Waiting doesn't diagnose the issue - sessions may not self-heal

**Module 3.2 Reference:** Concept 1 - Fabric Dashboard
</details>

---

#### Question 2: Interface Error Identification

**Scenario:** On the Interfaces Dashboard, you see this error counter:

```
Interface          CRC Errors   Frame Errors   Discards
leaf-01/Ethernet5  0            0              25,341
```

What is the most likely cause?

- A) Bad cable or dirty fiber
- B) Congestion causing packet drops
- C) Switch ASIC failure
- D) Incorrect VLAN configuration

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) Congestion causing packet drops

**Explanation:**

**Discard errors** (also called drops) indicate packets were intentionally dropped, typically due to:
- Output queue full (congestion)
- Rate limiting
- Buffer exhaustion

**CRC and Frame errors** indicate physical layer issues (bad cable, duplex mismatch).

Since CRC and Frame errors are **0**, the physical layer is fine. High **Discards** point to congestion.

**Next steps:**
1. Check Interface Utilization panel for Ethernet5
2. If utilization > 90%, congestion is confirmed
3. Check traffic rate over time - is there a sustained spike?
4. Investigate: Is server sending too much traffic? Is there a bandwidth limit needed?

**Why other options are wrong:**
- **A**: Bad cable would show CRC/Frame errors (not just discards)
- **C**: ASIC failure would affect multiple interfaces
- **D**: VLAN config errors prevent traffic, don't cause discards

**Module 3.2 Reference:** Concept 2 - Interfaces Dashboard (Error Counters)
</details>

---

#### Question 3: Hardware Health Assessment

**Scenario:** The Platform Dashboard shows:

```
Switch: leaf-03
PSU1: OK
PSU2: OK
Fan Tray 1: 5,200 RPM
Fan Tray 2: 0 RPM
CPU Temp: 55°C
ASIC Temp: 72°C
```

What action should you take?

- A) No action needed - one fan is sufficient
- B) Schedule immediate fan replacement - thermal risk
- C) Restart the switch to reset the fan
- D) Ignore fan failure until ASIC temp exceeds 80°C

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) Schedule immediate fan replacement - thermal risk

**Explanation:**

**Fan Tray 2 = 0 RPM** indicates a failed fan. Even though temperatures are currently OK (55°C CPU, 72°C ASIC), this is a **thermal risk**:

- Single fan failure reduces cooling redundancy
- If workload increases or ambient temp rises, switch may overheat
- Overheating can cause:
  - Automatic thermal shutdown
  - Reduced performance (thermal throttling)
  - Hardware damage

**Best Practice:**
- Failed fans should be replaced promptly (not during incident)
- Don't wait for temperatures to rise - proactive replacement
- Log the issue, create maintenance ticket

**Why other options are wrong:**
- **A**: Single fan is NOT sufficient - redundancy is critical
- **C**: Restart doesn't fix hardware failure (fan is broken)
- **D**: Waiting until overheating is reactive - replace proactively

**Module 3.2 Reference:** Concept 3 - Platform Dashboard (Fan Speeds)
</details>

---

#### Question 4: ASIC Resource Capacity

**Scenario:** The Switch Critical Resources Dashboard shows:

```
Switch: spine-01
IPv4 Route Table: 29,500 / 32,768 (90%)
IPv4 Neighbor (ARP): 8,000 / 16,384 (49%)
ACL Entries: 512 / 2,048 (25%)
```

What action should you take?

- A) No action - all resources below 100%
- B) Immediately reduce route count
- C) Monitor route table closely and plan for scale-out (add spine)
- D) Delete unused ARP entries to free resources

<details>
<summary>Answer & Explanation</summary>

**Answer:** C) Monitor route table closely and plan for scale-out (add spine)

**Explanation:**

**IPv4 Route Table at 90%** is a **warning state** - nearing ASIC hardware limit.

**Risk:** If route table fills to 100%:
- New routes cannot be installed
- BGP may fail to install prefixes
- Connectivity issues

**Action plan:**
1. **Immediate:** Monitor route table daily (track growth rate)
2. **Short-term:** Investigate why route count is high (route aggregation possible?)
3. **Long-term:** Plan for scale-out (add spine switches to distribute routes)
4. **Set alert:** Alert at 85% to catch before critical

**Why other options are wrong:**
- **A**: 90% is too close to limit - action required
- **B**: "Immediately reduce" may impact connectivity - plan carefully
- **D**: ARP at 49% is healthy - no action needed (wrong focus)

**Key principle:** ASIC resources are hardware limits - can't be increased on existing switch. Must plan for capacity.

**Module 3.2 Reference:** Concept 6 - Switch Critical Resources Dashboard
</details>

---

## Technical Requirements

### Grafana Dashboards

**Dashboard Access:**
- **Grafana URL:** http://localhost:3000
- **Authentication:** admin / prom-operator
- **Dashboards Location:** Dashboards menu (left sidebar)

**Dashboard List:**

1. **Hedgehog Fabric** - BGP and underlay health
2. **Hedgehog Interfaces** - Interface state and traffic
3. **Hedgehog Platform** - Hardware health
4. **Hedgehog Logs** - Syslog and error filtering
5. **Node Exporter** - Linux system metrics
6. **Switch Critical Resources** - ASIC resource capacity

**Dashboard UIDs** (for reference):
- Fabric: `ab831ceb-cf5c-474a-b7e9-83dcd075c218`
- Interfaces: `a5e5b12d-b340-4753-8f83-af8d54304822`
- Platform: `f8a648b9-5510-49ca-9273-952ba6169b7b`
- Logs: `c42a51e5-86a8-42a0-b1c9-d1304ae655bc`
- Node Exporter: `rYdddlPWA`
- Critical Resources: `fb08315c-cabb-4da7-9db9-2e17278f1781`

### Common Dashboard Operations

**Time Range Selection:**
```
- Last 5 minutes
- Last 15 minutes
- Last 1 hour (default for health checks)
- Last 24 hours (for trend analysis)
- Last 7 days (for weekly patterns)
- Custom range
```

**Refresh Rate:**
```
- Off (manual refresh)
- 5s, 10s, 30s (live monitoring)
- 1m, 5m (default)
```

**Panel Actions:**
- **View**: Click panel title → View
- **Edit**: Click panel title → Edit (requires edit permissions)
- **Share**: Click panel title → Share → Link (snapshot)
- **Explore**: Click panel title → Explore (ad-hoc PromQL queries)

**Filtering:**
- Most dashboards have "Switch" dropdown at top
- Select specific switch to filter all panels

### Reference Documents

- **[OBSERVABILITY.md](../research/OBSERVABILITY.md)** - Dashboard details (lines 296-401)
- **[Module 1.3 Design](./module-1.3-design.md)** - Three interfaces introduction

---

## Pedagogical Design

### Learning Philosophy Principles Emphasized

#### 1. **Focus on What Matters Most** ⭐
- **How:** Focuses on dashboards operators check daily (Fabric, Interfaces, Platform)
- **Example:** Morning health check workflow (5-minute daily routine)
- **Why:** Builds habits for proactive monitoring

#### 2. **Confidence Before Comprehensiveness** ⭐
- **How:** Systematic progression through dashboards (simple → complex)
- **Example:** BGP (binary up/down) before ASIC resources (capacity planning)
- **Why:** Early wins build confidence with dashboard interpretation

#### 3. **Learn by Doing, Not Watching** ⭐
- **How:** Hands-on health check workflow using live dashboards
- **Example:** Students perform actual morning health check
- **Why:** Muscle memory for daily operations

#### 4. **Train for Reality, Not Rote** ⭐
- **How:** Scenario-based interpretation (What does this error mean? What action?)
- **Example:** Assessment questions with realistic dashboard states
- **Why:** Prepares for real operational decisions

#### 5. **Teach the Why Behind the How** ⭐
- **How:** Explains what healthy/unhealthy states look like and why
- **Example:** Why 90% route table capacity is risky (hardware limit)
- **Why:** Enables independent decision-making

#### 6. **Support as Part of Learning**
- **How:** Module 3.4 teaches when/how to escalate with diagnostics
- **Example:** Identifying when to escalate vs self-resolve
- **Why:** Normalizes collaboration with support

---

### Target Audience Considerations

#### For Cloud-Native Learners

**What They Bring:**
- Grafana dashboard familiarity
- Prometheus metrics understanding
- Monitoring/alerting mindset

**What's New:**
- Network-specific metrics (BGP, interfaces)
- Hardware health metrics (PSU, fans, temperature)
- ASIC resource constraints

**Bridge Strategy:**
- Relate to Kubernetes pod/node dashboards (similar concepts)
- Frame BGP like service mesh health
- Compare ASIC limits to Kubernetes resource quotas

#### For Networking Professionals

**What They Bring:**
- Deep understanding of BGP, interfaces, switch hardware
- SNMP monitoring experience
- Physical layer troubleshooting

**What's New:**
- Grafana dashboard navigation
- Time-series visualization
- Prometheus-based metrics (vs SNMP)

**Bridge Strategy:**
- Compare to SNMP dashboards (similar goals, modern tools)
- Show how PromQL queries replace SNMP walks
- Frame Grafana as "SNMP NMS, but better"

---

## Dependencies

### Prerequisites (Must Complete First)

**Module 3.1:**
- ✅ Understanding of telemetry architecture
- ✅ Prometheus metrics and PromQL basics
- ✅ Metric types (counters, gauges)

**Why This Matters:**
- Dashboards visualize Prometheus metrics from Module 3.1
- Understanding metric types helps interpret panel queries
- PromQL knowledge enables dashboard customization

---

### Enables (Unlocks These Modules)

**Immediate:**
- ✅ **Module 3.3:** Events & Status Monitoring
  - Correlates Grafana metrics with kubectl events
  - Dashboard anomalies trigger kubectl investigation

**Sequential:**
- ✅ **Module 3.4:** Pre-Support Diagnostic Checklist
  - Dashboards inform which diagnostics to collect
  - Screenshots of dashboards included in support tickets

**Course-Level:**
- ✅ Foundation for Course 4 troubleshooting workflows
- ✅ Dashboards referenced in all future troubleshooting scenarios

---

## Quality Checklist

### Design Quality

- ✅ **Learning objectives are specific and measurable**
  - LO 1-6: Interpret each dashboard (testable via lab tasks)
  - LO 7: Create health check workflow (testable via lab completion)

- ✅ **Content outline follows logical progression**
  - Introduction → Core Concepts (6 dashboards) → Hands-On Lab (health check) → Assessment

- ✅ **Assessment aligns with learning objectives**
  - Question 1: Fabric Dashboard (BGP interpretation)
  - Question 2: Interfaces Dashboard (error types)
  - Question 3: Platform Dashboard (hardware health)
  - Question 4: Critical Resources (capacity planning)

- ✅ **Timing target is achievable (14-16 minutes)**
  - Introduction: 2 min
  - Core Concepts: 6-7 min (6 dashboards)
  - Hands-On Lab: 5-6 min (5 tasks)
  - Wrap-Up & Assessment: 2 min
  - **Total: 15-17 minutes** (slightly over 15 min due to 6 dashboards, acceptable)

### Technical Accuracy

- ✅ **All dashboard references validated against OBSERVABILITY.md**
  - Dashboard names and purposes match (lines 300-401)
  - Key panels documented accurately
  - UIDs correct (from Module 1.3 validation)

- ✅ **No technical errors or outdated information**
  - Grafana URL correct (localhost:3000)
  - Panel descriptions match actual dashboards
  - Healthy/unhealthy thresholds realistic

### Learning Philosophy

- ✅ **Embodies at least 3 core principles (embodies 6 of 10!)**
  - Focus on What Matters Most ⭐
  - Confidence Before Comprehensiveness ⭐
  - Learn by Doing, Not Watching ⭐
  - Train for Reality, Not Rote ⭐
  - Teach the Why Behind the How ⭐
  - Support as Part of Learning ⭐

- ✅ **Builds confidence through achievable goals**
  - Simple health check workflow
  - Clear healthy/unhealthy criteria
  - Immediate visual feedback from dashboards

### Completeness

- ✅ **All required sections filled out**
  - Module Metadata ✓
  - Learning Objectives ✓
  - Content Outline ✓
  - Hands-On Lab ✓
  - Assessment Design ✓
  - Technical Requirements ✓
  - Pedagogical Design ✓
  - Dependencies ✓
  - Quality Checklist ✓

---

## Document Metadata

**Created:** 2025-10-16
**Author:** Course Lead (Claude Code)
**Review Status:** ⏳ PENDING Course Lead Approval
**Version:** 1.0
**Previous Module:** Module 3.1 (Fabric Telemetry Overview - DESIGNED)
**Next Module:** Module 3.3 (Events & Status Monitoring - To Be Designed)

**Change Log:**
- 2025-10-16 v1.0: Initial design based on Issue #10 requirements and OBSERVABILITY.md

---

**Status:** ⏳ DESIGN COMPLETE - Ready for Review
**Related Issue:** GitHub Issue #10 - [DESIGN] Course 3: Observability & Fabric Health (Modules 3.1-3.4)
**Design Duration:** ~5 hours (as estimated in Issue #10)
