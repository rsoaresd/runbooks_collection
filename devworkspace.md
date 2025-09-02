# devworkspace-suspicious-activity

## Overview

The `devworkspace-suspicious-activity` alert is triggered when suspicious user activities are detected in a namespace that contains no workload manifests, indicating the user likely utilized a DevWorkspace environment for potentially unauthorized activities.

## AI Analysis Instructions

> **IMPORTANT FOR AI AGENTS:** This runbook is for **analysis and recommendation only**. You are in read-only investigation mode. DO NOT attempt to modify, patch, delete, or fix any resources. Your role is to diagnose the issue and provide specific remediation recommendations for human operators.

\## Investigation Workflow

Follow these steps to systematically investigate suspicious activities in DevWorkspace environments:

### Phase 1: DevWorkspace Discovery

#### Step 1: Locate DevWorkspace Custom Resources
**Objective:** Identify all DevWorkspace instances in the suspicious namespace.

**AI Investigation:**
- Use available tools to retrieve DevWorkspace Custom Resources (CRs) in the target namespace
- Focus on DevWorkspace CR names and creation timestamps
- Look for workspace status and current state

**What to analyze:**
- DevWorkspace CR names and creation timestamps
- Workspace status and current state
- Associated metadata and labels
- Associated pods and container information

#### Step 2: Analyze DevWorkspace Configuration
**Objective:** Examine DevWorkspace specifications for suspicious indicators.

**AI Investigation:**
- Retrieve complete DevWorkspace YAML configuration
- Examine workspace specifications for source repository references
- Look for example project templates or unusual settings
- Analyze resource limits and security contexts

**Critical:** If ANY remaining resources are found, proceed to Phase 2.

### Phase 2: Workspace Content Analysis

#### Step 3: Catalog Workspace Content
**Objective:** Get detailed information about workspace file system and activities.

**AI Investigation:**
- Access the DevWorkspace pod's terminal environment
- Perform comprehensive file system analysis using commands like ls -al
- Document directory structure and file inventory
- Do not skip this step - file contents are essential for proper analysis

**Example:** If workspace contains unusual files:
- List all files in the workspace
- Get complete file details for each suspicious file found

#### Step 4: Analyze Content Classification
**Objective:** Identify specific content types and potential security risks.

**AI Analysis:**
- Classify files as example/template vs. user-created content
- Document exact file paths and content descriptions
- Categorize content as:
  - Standard Kubernetes/workspace content
  - User-created legitimate content
  - Potentially suspicious or malicious content

### Phase 3: Repository and Activity Analysis

#### Step 5: Determine Finalizer Ownership
**Objective:** Understand what repositories and external sources are being used.

**AI Analysis:**
- Check resource annotations for repository information
- Analyze repository URL patterns (GitHub, GitLab, etc.)
- Identify whether repositories are:
  - Safe to access (known legitimate sources)
  - Require investigation (unknown or suspicious sources)
  - Need blocking (malicious or unauthorized sources)

## Expected AI Output

After completing the investigation, provide a comprehensive analysis report with:

### 1. Recommended Actions for Human Operators

**For Standard DevWorkspace Content:**
```bash
# Stop workspace after investigation
kubectl patch devworkspace <workspace-name> -n <namespace> \
  --type='merge' -p '{"spec":{"started":false}}'
```

**For Suspicious Content:**
- Document specific file paths and content found
- Recommend security team escalation if malicious content detected
- Suggest workspace termination and user access review

**For Unknown Repositories:**
- Recommend repository legitimacy verification before removal
- Suggest checking organizational security policies

### 2. Root Cause Analysis
- Specific DevWorkspace names and creation dates involved
- Exact repository URLs and external references found
- Why the activity appears suspicious (if determinable)

### 3. Current System State
- Complete inventory of workspace content
- Repository references and legitimacy assessment
- Any unusual configurations or behaviors

### 4. Prevention Recommendations
- Best practices for DevWorkspace usage monitoring
- Security policy updates for external repository access
- Process improvements for suspicious activity detection

## Example Scenarios

### Scenario 1: Resource with Suspicious Repository
```yaml
# If you find a DevWorkspace like this:
spec:
  template:
    components:
    - git:
        uri: https://github.com/suspicious-repo/malicious-tool
```

**AI Should Recommend:**
"The DevWorkspace references a suspicious external repository 'https://github.com/suspicious-repo/malicious-tool'. Recommend immediate workspace termination and security team investigation."

### Scenario 2: Multiple Suspicious Files
**AI Should Investigate:**
- Each file type and content separate
- Repository connections and legitimacy
- User access patterns and history

## Success Criteria

Investigation is complete when you can answer:
[ ] What DevWorkspace instances exist in the suspicious namespace?
[ ] What specific files or content appear suspicious?
[ ] What repository references are found and are they legitimate?
[ ] What exact commands should operators run to investigate further?
[ ] Are there any security risks or compliance violations identified?
