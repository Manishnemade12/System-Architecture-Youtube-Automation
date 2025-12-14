# Phase 10: Security Hardening and Worker Sandboxing

## What problem this phase solves
Protects the platform from malicious URLs, container escape, and lateral movement by isolating extraction/upload workloads and enforcing strict security controls.

## Why this phase is required at this stage
Extraction workers process untrusted content and run third-party tools (`yt-dlp`, `ffmpeg`). Security must be baked in before exposing the platform to production traffic.

## What exact features/components are implemented
- Pod security policies or runtime security profiles (AppArmor/SELinux).
- Dedicated sandboxed environments (gVisor, Kata Containers, Firecracker).
- Network policies preventing egress to unauthorized hosts (only known CDNs + YouTube).
- URL validation pipeline preventing SSRF/SSTI or malicious payloads.

## Technical design decisions
- Run each job in a constrained sandbox with limited CPU/memory.
- Use ephemeral pods that are destroyed after completion, preventing persistence of malicious code.
- Validate MIME types and file signatures before handing files to clients.

## Technologies, tools, services, and frameworks required
- Kubernetes PodSecurityPolicies or Gatekeeper policies (Open Policy Agent).
- Runtime defense (Falco) for suspicious syscalls.
- Network policy controller (Calico/Cilium) for egress restrictions.

## APIs, services, and integrations involved
- Security service integrates with logging (Phase 09) to surface policy violations.
- Authorization service ensures only validated tenants can spin up new jobs.

## Database/storage considerations
- Security audit logs stored in immutable S3 bucket for compliance (WORM). 
- Access logs correlated with job records for forensic analysis.

## Security considerations
- Enforce least privilege for every service account (KSA) used by workers.
- Rotate secrets frequently and store them in Vault/KMS.
- Validate OAuth redirect URIs to prevent authorization code interception.

## Performance & scalability considerations
- Apply resource limits so malicious jobs cannot starve the system.
- Use admission controllers to prevent overbooking nodes.

## Dependencies on previous phases
- Leverages identity/tenant data (Phase 02) and worker definitions (Phase 05-07).

## What output/deliverable this phase produces
- Hardened security baseline with documented policies and sandbox configurations.
- Security monitoring aligned with observability output.
