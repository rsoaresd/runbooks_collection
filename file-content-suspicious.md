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

### Phase 2: Kubernetes-Aware File System Investigation

#### Step 2: Analyze Pod and Container Context
**Objective:** Understand the container environment where suspicious content was detected.

**AI Investigation:**
- Extract pod information from the file path (e.g., `/proc/429630/` indicates process ID)
- Correlate process ID with running pods using available Kubernetes tools
- Analyze container specifications and image sources
- Check for volume mounts that could introduce suspicious files
- Review pod security context and capabilities
- **Attempt to read file contents if accessible through pod logs or mounted volumes**

**TARSy Limitations:**
- Cannot directly access files within pod containers
- Cannot execute into pods to read file contents
- Limited to Kubernetes API-accessible metadata and logs
- **Cannot use `oc exec` to cat file contents**

**Available Investigation Approaches:**
```bash
# Find specific pod information (when namespace is known)
oc get pods -n <namespace> -o wide

# If namespace is unknown, extract from file path context first
# Example: /proc/429630/root/... suggests checking pods by process/node
oc get pods -o wide --all-namespaces --field-selector spec.nodeName=<node-name>

# Analyze pod specifications
oc describe pod <pod-name> -n <namespace>

# Check container image and volume sources
oc get pod <pod-name> -n <namespace> -o yaml

# Check if file content appears in pod logs (limited cases)
oc logs <pod-name> -n <namespace> | grep -i "suspicious_pattern"

# Look for file references in container environment or config
oc get pod <pod-name> -n <namespace> -o jsonpath='{.spec.containers[*].env[*]}'
```

**Example Analysis:**
Source: /proc/429630/root/usr/lib/python3.6/site-packages/pgadmin4-web/pgadmin/static/scss/resources/dark
- Process ID: 429630 (correlate with running pods)
- Application: pgAdmin4 web interface (from path analysis)
- Content type: SCSS resources (styling files)
- Detected pattern: "dark" (likely dark theme files)

#### Step 2.5: Alternative Investigation When File Access is Blocked
**Objective:** Investigate suspicious content through Kubernetes metadata when direct file access is unavailable.

**Container Image Analysis:**
- Get pod container image details and sources
- Check image pull policies and registries
- Analyze image layers if scanning tools available
- Verify image signatures and scan results

**Volume and Mount Analysis:**
- Check mounted volumes and persistent volume claims
- Analyze config maps and secrets that might contain suspicious content
- Review volume sources and access modes
- Identify external volume mounts that could introduce files
- **Check if suspicious file is in a mounted ConfigMap or Secret**

**ConfigMap/Secret Content Analysis:**
```bash
# Check if file path matches mounted ConfigMaps (in known namespace)
oc get configmaps -n <namespace> -o yaml | grep -A5 -B5 "suspicious_filename"

# Check if file path matches mounted Secrets (in known namespace)
oc get secrets -n <namespace> -o yaml | grep -A5 -B5 "suspicious_filename"

# List all mounted volumes and their sources for specific pod
oc describe pod <pod-name> -n <namespace> | grep -A10 "Mounts:"
```

**Process and Network Analysis:**
- Check resource usage patterns for anomalies
- Analyze pod events for suspicious activities
- Review network policies and service connections
- Monitor pod restart patterns and failure reasons

#### Step 3: Assess Content Legitimacy Through Available Metadata
**Objective:** Determine if the detected content is legitimate using Kubernetes-accessible information.

**AI Investigation:**
- Research the application context from pod labels and annotations
- Analyze container image sources and known application patterns
- Check if detected pattern makes sense for the deployed application
- Review pod deployment history and configuration changes
- Cross-reference with known legitimate application file structures

**When Direct File Access is Required:**
- Document specific files that need manual inspection
- Prepare escalation details for security operators
- Identify alternative verification methods
- Note confidence level of assessment based on available metadata

### Phase 3: Risk Assessment

#### Step 4: Evaluate Potential Impact
**Objective:** Assess the security risk and potential impact of the detected content.

**AI Analysis:**
- Consider pod location sensitivity:
  - System namespaces (high risk)
  - Application namespaces (medium risk)
  - User namespaces (variable risk)
  - Temporary/job namespaces (lower risk)
- Analyze execution context:
  - Container privileges and capabilities
  - Service account permissions
  - Network access and policies
  - Volume mount permissions

#### Step 5: Check for Indicators of Compromise
**Objective:** Look for additional signs that may indicate actual malicious activity.

**AI Investigation:**
- Examine pod events and status changes
- Look for patterns indicating:
  - Recent unauthorized image updates
  - Unusual resource consumption
  - Suspicious network connections
  - Multiple related detections across pods
  - Abnormal pod restart patterns

## TARSy Tool Limitations for File Content Investigation

### Available Capabilities Through Kubernetes MCP Server
- ✅ Pod metadata and status analysis
- ✅ Container specifications and image details
- ✅ Volume mount configurations
- ✅ Pod logs and events analysis
- ✅ Resource usage patterns
- ✅ Network policy analysis
- ✅ Namespace and RBAC investigation
- ✅ Deployment and ReplicaSet analysis
- ✅ Service and Ingress configuration review

### Unavailable Capabilities
- ❌ Direct file system access within pods (`oc exec` not available)
- ❌ File content reading from containers
- ❌ Process execution inside pods
- ❌ File integrity verification (checksums)
- ❌ Real-time file monitoring
- ❌ Malware scanning of pod contents
- ❌ Container runtime security analysis

### Investigation Strategy
1. **Use TARSy for initial triage** - pod context, image analysis, volume investigation
2. **Escalate for direct access** - when file content inspection is required
3. **Coordinate with security tools** - integrate with runtime security monitoring

## Expected AI Output

After completing the investigation, provide a comprehensive analysis report with:

### 1. Detection Summary
Alert Type: file-content-suspicious
Detected Pattern: /dark
File Location: /proc/429630/root/usr/lib/python3.6/site-packages/pgadmin4-web/pgadmin/static/scss/resources/dark
Rule Matched: filename-suspicious (^|/)dark$
Threat Category: [Legitimate Application File | Potential False Positive | Requires Investigation | High Risk]

### 2. Kubernetes Context Analysis
**Pod Information:**
- Pod Name: pgadmin-deployment-xxx
- Namespace: database-tools
- Container Image: docker.io/dpage/pgadmin4:latest
- Image Source: Official registry
- Volume Mounts: [List relevant mounts]
- Security Context: [Privileges and capabilities]

### 3. Legitimacy Assessment
**For Legitimate Files:**
- Explain why the detection is a false positive based on Kubernetes context
- Document the legitimate purpose of the file/pattern within the application
- Reference container image documentation or known application structure
- Recommend rule tuning to prevent future false positives

**For Suspicious Files:**
- Detail specific indicators of malicious intent from available metadata
- Explain potential attack vectors or purposes
- Assess the confidence level of the threat assessment
- Note limitations due to inability to inspect file contents directly

### 4. Recommended Actions for Security Operators

**For False Positives:**
```bash
# Add exception to scanning rules for legitimate application patterns
# Update ruleset configuration to exclude known good applications
# Document in false positive database with Kubernetes context
```

**For Legitimate Suspicious Patterns:**
```bash
# Verify pod and container integrity
oc get pod <pod-name> -n <namespace> -o yaml
oc describe pod <pod-name> -n <namespace>

# Check deployment and image history
oc rollout history deployment/<deployment-name> -n <namespace>

# Manual file verification (requires escalation)
oc exec -it <pod-name> -n <namespace> -- sha256sum /path/to/file
```

**For Confirmed Threats:**
```bash
# Isolate the affected pod (coordinate with incident response)
oc cordon <node-name>
oc drain <node-name> --ignore-daemonsets

# Preserve evidence before remediation
oc get pod <pod-name> -n <namespace> -o yaml > evidence/pod-spec.yaml
oc logs <pod-name> -n <namespace> > evidence/pod-logs.txt

# Quarantine and analyze
oc scale deployment <deployment-name> --replicas=0 -n <namespace>
```

### 5. Root Cause Analysis
- Specific pod and container context involved
- Exact rules that triggered the alerts
- Why these patterns were flagged as suspicious
- Application context and legitimacy assessment based on Kubernetes metadata
- Confidence level of assessment given TARSy's limitations

### 6. Prevention Recommendations
- Rule tuning to reduce false positives for legitimate applications
- Application whitelisting for known good container images
- Enhanced monitoring for confirmed threat patterns
- Runtime security tool integration (Falco, Twistlock, etc.)
- Pod security policy or security context improvements

## Example Scenarios

### Scenario 1: Legitimate Application File (TARSy Analysis)
```yaml
detected: /dark
source: /proc/429630/root/usr/lib/python3.6/site-packages/pgladmin4-web/pgadmin/static/scss/resources/dark
context:
  matched-rule: (^|/)dark$
  ruleset-id: filename-suspicious
```

**AI Should Recommend:**
"Based on Kubernetes pod analysis, this appears to be a legitimate dark theme resource file for pgAdmin4 web interface deployed in pod `pgadmin-deployment-xxx`. Container image analysis shows official pgAdmin4 image from trusted registry. The 'dark' pattern is commonly used for UI theming in web applications.

**TARSy Investigation Results:**
- Pod: pgladmin-deployment-xxx in namespace: database-tools
- Image: docker.io/dpage/pgladmin4:latest (official)
- Volume mounts: No suspicious external volumes
- Resource usage: Normal patterns
- Security context: Non-privileged container
- **File content analysis: Unable to access file contents through Kubernetes API**

**Recommendation:** This is a false positive. Add exception for pgAdmin4 static resources to prevent future alerts."

**Manual Verification Required:** Security operators should verify file contents using:
```bash
oc exec -it pgladmin-deployment-xxx -n database-tools -- cat /usr/lib/python3.6/site-packages/pgladmin4-web/pgadmin/static/scss/resources/dark
```

### Scenario 2: Suspicious System File
```yaml
detected: /tmp/.dark_payload
source: /proc/12345/root/tmp/.dark_payload
context:
  matched-rule: (^|/)dark.*$
  ruleset-id: filename-suspicious
```

**AI Should Recommend:**
"Hidden file in /tmp with suspicious naming pattern detected in container process. Kubernetes analysis shows:

**TARSy Investigation Results:**
- Pod: unknown-workload-xxx in namespace: default
- Image: alpine:latest (generic base image - suspicious for production)
- Volume mounts: Host /tmp mounted (HIGH RISK)
- Security context: Privileged container (HIGH RISK)
- Recent events: Multiple pod restarts
- **File content analysis: Cannot access due to TARSy limitations**

**Risk Assessment:** HIGH - Combination of generic image, privileged access, host volume mounts, and suspicious file pattern indicates potential compromise.

**Immediate Actions Required:** 
1. Isolate the pod immediately
2. **Extract and analyze file contents immediately:**
   ```bash
   oc exec -it unknown-workload-xxx -n default -- cat /tmp/.dark_payload
   oc exec -it unknown-workload-xxx -n default -- file /tmp/.dark_payload
   oc cp default/unknown-workload-xxx:/tmp/.dark_payload ./evidence/dark_payload
   ```
3. Check for lateral movement to other pods
4. Review how this pod was deployed"

### Scenario 3: Investigation Limited by TARSy Capabilities
```yaml
detected: /malware_sample
source: /proc/98765/root/opt/app/malware_sample
context:
  matched-rule: malware.*
  ruleset-id: filename-suspicious
```

**AI Should Recommend:**
"Potential malware file detected in application container. TARSy analysis limited by read-only Kubernetes access.

**Available Analysis:**
- Pod: app-server-xxx in namespace: production
- Image: custom-app:v1.2.3 (private registry)
- Volume mounts: Application data volume
- Security context: Standard non-privileged
- **File content: REQUIRES MANUAL EXTRACTION**

**Limitations:** Cannot verify actual file contents or perform malware analysis through Kubernetes API.

**Required Escalation:** 
1. **Security team must immediately extract file contents:**
   ```bash
   oc exec -it app-server-xxx -n production -- cat /opt/app/malware_sample
   oc exec -it app-server-xxx -n production -- hexdump -C /opt/app/malware_sample | head -50
   oc cp production/app-server-xxx:/opt/app/malware_sample ./malware-analysis/
   ```
2. Perform offline malware analysis on extracted file
3. Consider pod quarantine pending investigation results
4. Review container image build process for compromise"

### Scenario 4: File Content Available Through ConfigMap
```yaml
detected: /config/suspicious.conf
source: /proc/55555/root/etc/app/config/suspicious.conf
context:
  matched-rule: suspicious.*
  ruleset-id: filename-suspicious
```

**AI Should Recommend:**
"Suspicious configuration file detected. TARSy can analyze this through mounted ConfigMap.

**TARSy Investigation Results:**
- Pod: web-app-xxx in namespace: applications
- Image: nginx:1.21 (official)
- Volume mounts: config-volume from ConfigMap 'app-config'
- **File content accessible through ConfigMap:**

```bash
oc get configmap app-config -n applications -o yaml
```

**ConfigMap Analysis:**
- ConfigMap contains suspicious.conf with content: [CONTENT WOULD BE VISIBLE]
- File appears to be legitimate application configuration
- No malicious patterns detected in accessible content

**Recommendation:** Legitimate configuration file. Consider renaming to avoid false positives."

## Success Criteria

Investigation is complete when you can answer:
- [ ] Is the detected content legitimate or malicious based on available Kubernetes metadata?
- [ ] **Have you attempted to access file contents through all available Kubernetes resources (ConfigMaps, Secrets, logs)?**
- [ ] What is the specific threat level and potential impact within the container context?
- [ ] What immediate actions should security operators take?
- [ ] Are there related pods or deployments that need investigation?
- [ ] Should detection rules be tuned to prevent false positives for this application?
- [ ] **What specific commands are needed to extract and analyze the file contents?**
- [ ] What manual investigation steps are required due to TARSy's limitations?
- [ ] Is immediate pod isolation or quarantine recommended?
