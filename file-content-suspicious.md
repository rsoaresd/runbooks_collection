# file-content-suspicious

## Overview

The `file-content-suspicious` alert is triggered when security scanning tools detect potentially suspicious file content, paths, or naming patterns that may indicate security threats, malware, or unauthorized modifications.

## AI Analysis Instructions

> **IMPORTANT FOR AI AGENTS:** This runbook is for **analysis and recommendation only**. You are in read-only investigation mode. DO NOT attempt to modify, delete, quarantine, or remediate any files. Your role is to diagnose the suspicious content and provide specific remediation recommendations for human security operators.

## Investigation Workflow

Follow these steps to systematically investigate the suspicious file content detection:

### Phase 1: Alert Context Analysis

#### Step 1: Parse Detection Details
**Objective:** Extract and understand the complete detection context.

**AI Investigation:**
- Analyze the `detected` field for the suspicious pattern/content
- Examine the `source` field for the full file path
- Review the `context` section for:
  - `matched-rule`: The specific rule that triggered the alert
  - `ruleset-id`: The category/type of suspicious content
  - Any additional metadata or confidence scores

**What to analyze:**
- File path components and directory structure
- File extension and naming patterns
- Rule type and severity level
- Detection timestamp and scanning tool information

#### Step 2: Categorize Threat Type
**Objective:** Classify the type of suspicious content detected.

**AI Analysis:**
- Map `ruleset-id` to threat categories:
  - `filename-suspicious`: Potentially malicious file names
  - `content-malware`: Known malware signatures
  - `config-backdoor`: Suspicious configuration changes
  - `script-obfuscated`: Obfuscated or encoded scripts
  - `binary-packed`: Packed or encrypted executables
- Assess threat severity based on rule type and file location

### Phase 2: File System Investigation

#### Step 3: Analyze File Location Context
**Objective:** Understand the file's location and legitimacy within the system.

**AI Investigation:**
- Examine the full file path for:
  - System vs. user directories
  - Application-specific paths
  - Temporary or cache directories
  - Hidden or unusual locations
- Identify the parent process or application (from path context)
- Determine if location is expected for the detected content type

**Example Analysis:**
