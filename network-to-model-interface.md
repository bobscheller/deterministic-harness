**Sample OpenShell Deployment on Google Kubernetes Engine**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: openshell-policy-config
  namespace: openshell
data:
  policy.yaml: |
    version: "1.0"
    sandbox:
      filesystem:
        allowed_reads:
          - /sandbox
          - /tmp
        allowed_writes:
          - /sandbox/output
          - /tmp
      network:
        allowed_egress:
          # Whitelist Google Vertex AI endpoints on port 443
          - host: "us-central1-aiplatform.googleapis.com"
            port: 443
          - host: "aiplatform.googleapis.com"
            port: 443
      process:
        allow_exec: true
        max_processes: 16
```
Additionally, OpenShell should be configured with a baseline policy and modified to support the system requirements.

**Sample OpenShell Configuration**

```yaml
version: "1.0"

# 1. Sandbox Environment Definition
sandbox:
  # Restrict the container's view of the host filesystem
  filesystem:
    allowed_reads:
      - /app/runtime          # Allow reading application code
      - /etc/ssl/certs        # Required for HTTPS/Vertex AI calls
      - /usr/share/zoneinfo   # Timezone data
    allowed_writes:
      - /tmp/agent_logs       # Minimal scratch space
      - /sandbox/output       # Specific directory for agent results
    block_sensitive_paths: true # Blocks /proc, /sys, /etc/shadow by default

  # 2. Strict Network Control
  network:
    default_action: "BLOCK"   # Deny all traffic unless explicitly allowed
    allowed_egress:
      # Specifically allow connection to Gemini Flash via Vertex AI
      - host: "us-central1-aiplatform.googleapis.com"
        port: 443
        protocol: "TCP"
      # Allow DNS resolution
      - host: "8.8.8.8"
        port: 53
        protocol: "UDP"

  # 3. Execution & Resource Governance
  process:
    allow_exec: true          # Allow the agent to run code (Python/Bash)
    max_processes: 10         # Prevent fork bombs
    max_memory_mb: 512        # Stop resource exhaustion attacks
    user_id: 1000             # Run as non-root user
    group_id: 1000

  # 4. Security Capabilities
  capabilities:
    drop:
      - ALL                   # Drop all Linux capabilities (NET_RAW, SYS_ADMIN, etc.)
    add:
      - CHOWN                 # Add back only what is strictly necessary
```