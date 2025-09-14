# Device Roles

Jason's three-node architecture distributes security functions across specialized hardware to eliminate conflicting requirements and single points of failure. Each device addresses a specific aspect of the **trust establishment problem** that characterizes IoT network security.

## NanoPi R1S — Access Plane: The Trust Gatekeeper
**Platform**: OpenWrt-based wireless access point  
**Security mandate**: Authenticate everything, trust nothing

The R1S confronts the fundamental challenge of wireless security: **how to simultaneously provide network connectivity and prevent network access until authorization is verified**. Traditional access points grant network privileges based on successful authentication, but this creates a window where partially-authenticated devices can attack each other or the infrastructure.

**Key security functions**:
- **WPA3-Personal (SAE) with Management Frame Protection** eliminates the cryptographic vulnerabilities that have plagued Wi-Fi security for decades, ensuring that wireless association requires proof of shared secret knowledge
- **RADIUS-integrated MAC ACL workflow** binds device identity to network privileges, preventing unauthorized devices from accessing the network even if they possess valid credentials
- **Per-client Layer-2 isolation** creates individual broadcast domains for each connected device, eliminating the attack surface where compromised devices pivot to legitimate ones during authentication
- **Kernel and sysctl hardening** reduces the device's exposure to denial-of-service attacks and reconnaissance probes that target wireless infrastructure

The R1S operates on the principle that **proximity to wireless infrastructure does not imply permission to use network resources**. Every device must cryptographically prove its authorization before receiving any network privileges.

## NanoPi R2S — Segmentation Plane: The Policy Enforcer  
**Platform**: IPFire zone-based firewall  
**Security mandate**: Make unauthorized communication architecturally impossible

The R2S implements the core security hypothesis of Jason: **network security failures result from policy-implementation gaps, not inadequate policy specification**. Rather than relying on administrative controls that can be bypassed or misconfigured, the R2S compiles security policy directly into kernel-enforced packet filtering.

**Key security functions**:
- **Zone-based firewall architecture** (GREEN/ORANGE/BLUE/RED) expresses security policy as spatial relationships between trust domains, making policy intent explicit and mathematically verifiable
- **Default-deny enforcement** ensures that communication between zones requires explicit justification—if a connection cannot be explained by legitimate device functionality, it cannot occur
- **Interface binding and Ethernet zoning** physically constrains traffic flows, preventing software-based attacks from bypassing network policy through interface manipulation
- **Stateful filtering with scoped ACLs** tracks connection state across security boundaries while maintaining minimal privilege sets for each authorized flow

The R2S transforms network policy from **administrative guidance into physical law**—unauthorized communications are not merely prohibited, they are structurally impossible within the network architecture.

## Raspberry Pi — Services Plane: The Intelligence Engine
**Platform**: Containerized security services stack  
**Security mandate**: Convert attacker activity into defender advantage

The Pi addresses the **information asymmetry problem** in IoT security: attackers know when they compromise devices, but defenders typically remain unaware until significant damage occurs. Rather than waiting for attacks to succeed, the Pi implements active measures that force attackers to reveal themselves.

**Key security functions**:
- **Unbound DNS resolver** with DNSSEC validation and QNAME minimization eliminates DNS as an attack vector while providing cryptographically verified resolution that can be trusted for security decisions
- **DNS sinkhole with Response Policy Zones** transforms threat intelligence into real-time protection, automatically redirecting malicious domains to controlled analysis environments
- **Distributed honeypot services** (SSH/HTTP/ICS protocols) present attractive targets that legitimate devices have no reason to contact, converting reconnaissance into high-confidence attack indicators  
- **CrowdSec behavioral analysis** enables community-driven threat detection, sharing attack signatures across Jason deployments for distributed immune response
- **ClamAV content scanning** provides signature-based detection for known malware families attempting to propagate through network services

The Pi operates on the insight that **attackers reveal their presence through behaviors that systematically differ from legitimate device operations**. By creating controlled environments where these differences become observable, the Pi transforms attacker reconnaissance into defender intelligence.

## Architectural Interdependence

These three devices function as **mutually reinforcing security controls**:

- The **R1S ensures that only authenticated devices enter the network**, but cannot determine what authenticated devices should be permitted to do
- The **R2S enforces what authenticated devices are permitted to do**, but cannot detect when devices exceed their behavioral baselines  
- The **Pi detects when devices exhibit suspicious behaviors**, but cannot enforce network policy restrictions

This interdependence eliminates single points of failure while ensuring that the compromise of any individual device cannot disable the entire security architecture. The result is **graceful security degradation** rather than catastrophic failure when components are attacked or misconfigured.