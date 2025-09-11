# cpu-consumption-high

## Overview

The `cpu-consumption-high` alert is triggered when CPU utilization exceeds defined thresholds, indicating potential performance bottlenecks, resource contention, or runaway processes that may impact system stability and application performance.

## AI Analysis Instructions

> **IMPORTANT FOR AI AGENTS:** This runbook is for **analysis and recommendation only**. You are in read-only investigation mode. DO NOT attempt to kill processes, modify system configurations, or restart services. Your role is to diagnose the CPU consumption issue and provide specific remediation recommendations for human operators.

## Investigation Workflow

Follow these steps to systematically investigate high CPU consumption:

### Phase 1: Alert Context Analysis

#### Step 1: Parse CPU Metrics
**Objective:** Extract and understand the complete CPU utilization context.

**AI Investigation:**
- Analyze the `cpu_percent` field for current utilization levels
- Examine the `threshold` field for the configured alert threshold
- Review the `duration` field for how long the condition has persisted
- Check `node` or `instance` field for the affected system
- Look for `process` or `container` specific metrics if available

**What to analyze:**
- Current CPU percentage vs. threshold
- Duration of high CPU condition
- System-wide vs. process-specific consumption
- Multi-core vs. single-core bottlenecks

### Phase 2: System Resource Investigation

#### Step 2: Analyze CPU Usage Patterns
**Objective:** Understand the CPU consumption characteristics and identify contributing factors.

**AI Investigation:**
- Examine CPU usage breakdown:
  - User space vs. kernel space consumption
  - I/O wait time percentages
  - System interrupt handling load
  - Idle time availability
- Identify load distribution across CPU cores
- Check for CPU steal time (in virtualized environments)

**Example Analysis:**
CPU Usage: 95% (threshold: 80%)
User: 70%, System: 20%, IOWait: 5%, Idle: 5%
Load Average: 8.5, 7.2, 6.8 (4-core system)
Duration: 15 minutes

#### Step 3: Identify Top CPU Consumers
**Objective:** Determine which processes or containers are consuming the most CPU resources.

**AI Investigation:**
- List top CPU-consuming processes with:
  - Process ID (PID) and parent process ID (PPID)
  - Process name and command line arguments
  - CPU percentage and memory usage
  - Process start time and duration
- For containerized environments, identify:
  - Container names and images
  - Kubernetes pod and namespace information
  - Resource limits and requests vs. actual usage

### Phase 3: Root Cause Analysis

#### Step 4: Analyze Process Behavior
**Objective:** Understand why specific processes are consuming excessive CPU.

**AI Investigation:**
- Examine process characteristics:
  - Is this expected behavior for the application?
  - Recent changes in process CPU patterns
  - Correlation with application load or user activity
  - Signs of infinite loops or runaway processes
- Check for:
  - Memory leaks causing excessive garbage collection
  - Inefficient algorithms or database queries
  - Resource contention or lock contention
  - External dependencies causing processing delays

#### Step 5: System-Level Analysis
**Objective:** Identify system-level factors contributing to high CPU usage.

**AI Investigation:**
- Check system health indicators:
  - Memory pressure causing swapping
  - Disk I/O bottlenecks affecting CPU wait
  - Network activity and interrupt handling
  - System daemon or kernel process consumption
- Examine resource limits and quotas:
  - Container CPU limits and throttling
  - Process nice values and scheduling priority
  - CPU affinity and NUMA topology effects

### Phase 4: Impact Assessment

#### Step 6: Evaluate Performance Impact
**Objective:** Assess the impact of high CPU usage on system and application performance.

**AI Analysis:**
- System responsiveness:
  - Load average trends and queue lengths
  - Response time degradation for critical services
  - User-facing application performance metrics
- Resource starvation effects:
  - Other processes being starved of CPU time
  - Critical system processes being impacted
  - Cascading effects on dependent services

#### Step 7: Check for Related Issues
**Objective:** Identify correlated problems that may be causing or caused by high CPU usage.

**AI Investigation:**
- Look for related alerts or metrics:
  - Memory usage spikes
  - Disk I/O saturation
  - Network congestion
  - Application error rates
- Check for recent changes:
  - Application deployments
  - Configuration changes
  - Infrastructure modifications
  - Traffic pattern changes

## Expected AI Output

After completing the investigation, provide a comprehensive analysis report with:

### 1. CPU Consumption Summary
Alert Type: cpu-consumption-high
Current CPU Usage: 95% (threshold: 80%)
Duration: 15 minutes
Affected System: web-server-01
Load Average: 8.5/7.2/6.8 (4-core system)
Primary Cause: [Application Process | System Process | Resource Contention | Unknown]

### 2. Top CPU Consumers Analysis
Top Processes:
1. java (PID: 12345) - 45% CPU - Application server
2. mysqld (PID: 23456) - 25% CPU - Database server  
3. nginx (PID: 34567) - 15% CPU - Web server
4. kworker (PID: 45678) - 10% CPU - Kernel worker

Container Analysis (if applicable):
- app-container: 60% CPU (limit: 2 cores, no throttling)
- db-container: 30% CPU (limit: 1 core, throttling detected)

### 3. Root Cause Assessment
**For Application Issues:**
- Identify specific application causing high CPU
- Explain likely causes (inefficient code, high load, resource leaks)
- Assess whether this is expected behavior or a problem

**For System Issues:**
- Detail system-level problems (I/O wait, interrupts, swapping)
- Explain how system issues are affecting CPU usage
- Identify infrastructure or configuration problems

**For Resource Contention:**
- Describe competing processes or containers
- Explain resource limit conflicts or misconfigurations
- Assess scheduling and priority issues

### 4. Recommended Actions for Operations Team

**For Application Performance Issues:**
```bash
# Check application logs for errors or performance issues
tail -f /var/log/application.log

# Analyze application performance metrics
# Review database query performance
# Check for memory leaks or inefficient algorithms

# Scale application horizontally if needed
kubectl scale deployment app-deployment --replicas=5
```

**For System Resource Issues:**
```bash
# Check system resource utilization
iostat -x 1 5
vmstat 1 5
sar -u 1 5

# Examine memory usage and swapping
free -h
swapon -s

# Review system logs for errors
journalctl -u systemd --since "1 hour ago"
```

**For Process Management:**
```bash
# Identify resource-intensive processes
top -c
htop
ps aux --sort=-%cpu | head -20

# Check process limits and priorities
cat /proc/PID/limits
renice -n 10 -p PID  # Lower priority for non-critical processes
```

**For Container/Kubernetes Issues:**
```bash
# Check container resource usage and limits
kubectl top pods --all-namespaces
kubectl describe pod POD_NAME

# Review resource quotas and limits
kubectl get resourcequota
kubectl describe node NODE_NAME

# Check for CPU throttling
kubectl get --raw /api/v1/nodes/NODE_NAME/proxy/metrics/cadvisor | grep throttl
```

### 5. Immediate vs. Long-term Actions

**Immediate (< 5 minutes):**
- Identify if critical services are affected
- Check if automatic scaling is working
- Verify system stability and responsiveness
- Determine if manual intervention is needed

**Short-term (< 1 hour):**
- Scale affected applications if possible
- Adjust process priorities for non-critical workloads
- Implement temporary resource limits
- Monitor for improvement or degradation

**Long-term (planning):**
- Capacity planning based on usage trends
- Application performance optimization
- Infrastructure scaling recommendations
- Resource allocation tuning

### 6. Prevention Recommendations
- **Monitoring Enhancements:** Set up predictive alerting based on CPU trends
- **Resource Planning:** Implement proper CPU limits and requests for containers
- **Application Optimization:** Profile and optimize high-CPU applications
- **Auto-scaling:** Configure horizontal pod autoscaling based on CPU metrics
- **Load Testing:** Regular performance testing to identify bottlenecks before production

## Example Scenarios

### Scenario 1: Application Load Spike
```yaml
cpu_percent: 85
threshold: 80
duration: 5m
node: web-server-01
top_process: java (spring-boot-app) - 70% CPU
```

**AI Should Recommend:**
"High CPU usage caused by increased application load on Spring Boot application. Check if this correlates with traffic spike. Recommend enabling horizontal pod autoscaling and reviewing application performance metrics. Monitor for memory leaks if sustained."

### Scenario 2: Database Query Performance
```yaml
cpu_percent: 92
threshold: 80  
duration: 20m
node: db-server-01
top_process: mysqld - 85% CPU
iowait: 15%
```

**AI Should Recommend:**
"Database server experiencing high CPU with significant I/O wait. Likely caused by inefficient queries or missing indexes. Recommend analyzing slow query log, checking for long-running transactions, and reviewing recent database schema changes."

### Scenario 3: System Resource Contention
```yaml
cpu_percent: 95
threshold: 80
duration: 30m
node: worker-node-02
load_average: [12.5, 10.2, 8.7]
cpu_cores: 4
```

**AI Should Recommend:**
"Severe CPU overload with load average 3x the number of cores. System is likely experiencing resource starvation. Recommend immediate load balancing to other nodes, checking for runaway processes, and reviewing resource allocation policies."

## Success Criteria

Investigation is complete when you can answer:
- [ ] What is causing the high CPU consumption?
- [ ] Is this expected behavior or a performance problem?
- [ ] What is the impact on system and application performance?
- [ ] What immediate actions are needed to restore normal operation?
- [ ] What long-term changes are needed to prevent recurrence?
- [ ] Are there related resource issues that need attention?
