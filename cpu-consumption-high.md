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
