# Data Flow

This security model becomes clear when tracing how data moves through the system—or more precisely, how the architecture **controls and constrains** that movement. Each packet traverses multiple security boundaries, with different enforcement mechanisms at each layer creating **defense in depth** that cannot be bypassed through single-point exploitation.

## Device Onboarding: Trust Through Cryptographic Proof

**The Challenge**: How do you safely integrate an unknown device into a network without creating attack opportunities?

When a new IoT device attempts network access, it encounters Jason's **trust establishment pipeline**:

1. **Wireless association attempt** triggers WPA3-SAE authentication on the NanoPi R1S, requiring cryptographic proof of shared secret knowledge before any network frames are processed
2. **MAC address validation** against RADIUS ACL occurs immediately after successful authentication—even devices with valid Wi-Fi credentials are rejected if their hardware identity is not pre-authorized
3. **Layer-2 isolation enforcement** places approved devices into individual broadcast domains, preventing lateral communication during the vulnerable post-authentication window
4. **VLAN tagging and zone assignment** routes the device into the appropriate security context (typically ORANGE zone) based on its authenticated identity

The critical insight: **authentication success does not equal network privilege escalation**. The device proves its right to exist on the network, but not its right to communicate freely.

## Inter-Zone Communication: Policy as Physical Law

**The Challenge**: How do you prevent legitimate devices from becoming attack vectors for lateral movement?

IoT devices cannot communicate directly with each other or with trusted network segments—all inter-zone traffic must traverse the NanoPi R2S segmentation plane, where **zone-based policies are mathematically enforced**:

- **Default-deny compilation** ensures that communication requests not explicitly permitted by policy are architecturally impossible, not merely administratively forbidden
- **Stateful connection tracking** maintains the complete context of authorized flows, preventing protocol-level attacks that abuse connection establishment
- **Source and destination validation** confirms that traffic originates from authenticated devices and targets only authorized services

This creates a **mathematical guarantee**: compromised IoT devices cannot reach unauthorized network segments because the communication paths simply do not exist within the network topology.

## Upstream Communication: DNS as Security Enforcement Point

**The Challenge**: How do you maintain device functionality while preventing communication with malicious infrastructure?

When IoT devices require external connectivity—DNS resolution, cloud API calls, firmware updates—their requests traverse the services plane for **cryptographic validation and threat analysis**:

- **DNSSEC validation through Unbound** ensures that DNS responses are cryptographically authentic, eliminating DNS poisoning as an attack vector
- **Response Policy Zone (RPZ) filtering** automatically redirects requests for known-malicious domains to controlled sinkhole environments, preventing communication with command-and-control infrastructure
- **Query pattern analysis** identifies anomalous DNS behavior that may indicate device compromise or reconnaissance activity

The services plane operates on the principle that **legitimate device communication has predictable patterns**, while attack traffic exhibits detectable anomalies in timing, destinations, and protocol usage.

## Threat Detection: Converting Reconnaissance into Intelligence

**The Challenge**: How do you detect sophisticated attacks that mimic legitimate device behavior?

Jason implements **active defense** through strategic deployment of deceptive services that convert attacker reconnaissance into high-confidence detection events:

- **Distributed honeypot services** present attractive targets (SSH, HTTP, industrial control system protocols) that legitimate IoT devices have no reason to contact
- **Behavioral correlation through CrowdSec** identifies attack patterns across multiple detection sources, enabling community-driven threat intelligence sharing
- **Real-time policy adaptation** allows the system to dynamically restrict devices exhibiting suspicious behavior without requiring manual intervention

This approach transforms the **information asymmetry** that typically favors attackers—instead of waiting to detect successful compromises, Jason forces attackers to reveal themselves through reconnaissance activity.

## Monitoring and Adaptive Response: Continuous Security Improvement

**The Challenge**: How do you maintain security posture as threats evolve and device populations change?

Jason implements a **feedback loop** that continuously improves security through systematic observation and policy refinement:

- **Centralized log correlation** aggregates security events from all three planes, providing comprehensive visibility into device behavior patterns
- **Per-device behavioral baselines** enable detection of anomalous activity that may indicate compromise or misconfiguration  
- **Automated policy suggestions** based on observed traffic patterns help administrators refine security policies without compromising device functionality
- **Research data collection** supports systematic evaluation of security policy effectiveness and threat model validation

## Security Flow Architecture

The data flow demonstrates Jason's **tiered security enforcement model**:

1. **Access Plane (R1S)**: Authenticates device identity and establishes initial trust boundaries
2. **Segmentation Plane (R2S)**: Enforces communication policies and prevents unauthorized lateral movement  
3. **Services Plane (Pi)**: Validates external communications and detects anomalous behavior

Each layer **assumes the potential failure of other layers** while providing independent security value. Device compromise cannot cascade through the system because each security boundary operates through different mechanisms with different attack surfaces.

This architecture demonstrates that **enterprise-grade network security through systematic application of fundamental security principles** rather than reliance on expensive specialized hardware or proprietary security solutions.