# Threat Model

This threat model defines the adversarial landscape that drives architectural decisions throughout the platform. Each control mechanism directly addresses specific attack vectors enumerated below, enabling systematic evaluation of defensive effectiveness.

## Adversary Taxonomy

### Opportunistic IoT Attackers
These adversaries exploit widespread vulnerabilities in commodity IoT devices through automated scanning and exploitation frameworks. **Attack vectors include:**

- **Credential-based attacks**: Default passwords, credential stuffing against web interfaces, and exploitation of hardcoded authentication tokens embedded in firmware
- **Protocol exploitation**: Abuse of insecure protocols (Telnet, HTTP management interfaces, MQTT without authentication) and implementation flaws in embedded TCP/IP stacks
- **Firmware vulnerabilities**: Memory corruption bugs, command injection in web interfaces, and privilege escalation through local services
- **Network service enumeration**: Port scanning, service fingerprinting, and exploitation of unnecessary exposed services (SSH, web servers, custom protocols)

**Attacker capabilities**: Automated tools, public exploit databases, basic network reconnaissance. **Typical goals**: Botnet recruitment, cryptocurrency mining, DDoS participation, data exfiltration for sale.

### Curious Insiders and Misconfigured Devices
This threat category encompasses both malicious insiders with legitimate network access and devices that unintentionally bridge security domains through misconfiguration or design flaws.

- **Trust boundary violations**: Smart TVs that discover and attempt to access file shares, voice assistants that leak network topology through mDNS broadcasts, and media players that attempt SMB connections to discover content
- **Credential sharing**: Devices that store and reuse network credentials across multiple services, enabling privilege escalation
- **Protocol bridging**: Devices that implement multiple network protocols (Wi-Fi, Bluetooth, Zigbee) and inadequately isolate traffic between them
- **Configuration drift**: Legitimate devices that accumulate insecure configurations over time through automatic updates or user modifications

**Attacker capabilities**: Legitimate network access, knowledge of internal network topology, potential physical access to devices. **Goals**: Data access beyond intended scope, network reconnaissance, persistent access establishment.

### Active Reconnaissance and Targeted Attackers
Sophisticated adversaries conducting deliberate campaigns against specific targets, combining automated tools with manual analysis and custom exploit development.

- **Multi-stage reconnaissance**: Network topology mapping, device fingerprinting, vulnerability correlation, and identification of high-value targets
- **Credential attacks**: Targeted password attacks using device-specific wordlists, certificate-based authentication bypass, and exploitation of weak cryptographic implementations
- **Lateral movement techniques**: Exploitation of trust relationships, abuse of administrative protocols, and privilege escalation through service misconfigurations
- **Persistence mechanisms**: Implantation of backdoors in device firmware, exploitation of automatic update mechanisms, and establishment of command-and-control channels through legitimate protocols

**Attacker capabilities**: Custom tooling, manual analysis, vulnerability research, potential zero-day exploits. **Goals**: Persistent access, data exfiltration, supply chain compromise, infrastructure disruption.

### Network and Metadata Observers
Passive adversaries who monitor network traffic and DNS queries to infer sensitive information without directly compromising devices.

- **DNS surveillance**: Monitoring queries to infer device types, user behavior patterns, and network topology
- **Traffic analysis**: Statistical analysis of encrypted communications to identify protocols, timing patterns, and communication relationships
- **Supply-side tracking**: Correlation of device behavior with manufacturer telemetry to build device and user profiles
- **Metadata aggregation**: Cross-correlation of network metadata with external data sources for enhanced surveillance capabilities

**Attacker capabilities**: Network monitoring, traffic analysis tools, correlation with external databases. **Goals**: Surveillance, behavioral profiling, intelligence gathering.

## Security Objectives

### Primary: Containment and Blast Radius Limitation
**Objective**: Architectural prevention of lateral movement following device compromise.

**Technical requirements**:
- L2 isolation preventing ARP spoofing and broadcast domain attacks
- L3 segmentation with stateful firewall enforcement  
- Application-layer filtering preventing protocol tunneling
- Identity binding preventing address-based privilege escalation

**Evaluation criteria**: Controlled penetration testing scenarios, formal verification of network reachability constraints, systematic enumeration of potential bypass techniques.

#### DNS Integrity and Privacy Assurance  
**Objective**: Eliminate DNS as an attack vector and information leakage channel.

**Technical requirements**:
- DNSSEC validation for all external queries
- Response Policy Zone (RPZ) implementation for real-time threat intelligence integration
- Query logging and analysis for behavioral anomaly detection
- Recursive resolution to prevent upstream query observation

**Evaluation criteria**: DNS spoofing resistance testing, measurement of metadata leakage reduction, analysis of threat intelligence integration effectiveness.

#### High-Signal Threat Detection
**Objective**: Early detection of reconnaissance and compromise attempts with minimal false positive rates.

**Technical requirements**:
- Distributed honeypot deployment mimicking legitimate services
- Behavioral analysis of network communications patterns
- Integration of community-driven threat intelligence
- Correlation engine for multi-source event analysis

**Evaluation criteria**: Detection timing analysis under controlled attack scenarios, false positive rate characterization across different device types and usage patterns, comparison with baseline detection approaches.

#### Comprehensive Auditability
**Objective**: Complete reconstruction of security events for incident response and forensic analysis.

**Technical requirements**:
- Structured logging of all network flows, authentication events, and policy violations
- Cryptographic log integrity protection
- Long-term log retention with efficient search capabilities  
- Automated correlation and timeline reconstruction

**Evaluation criteria**: Attack chain reconstruction completeness in simulated scenarios, log integrity verification, forensic timeline accuracy assessment.

## Assumptions and Scope Limitations

#### Physical Security Assumptions
The platform assumes physical security of all network infrastructure components. **Out of scope**: Hardware tampering, side-channel attacks against cryptographic implementations, physical theft of devices containing cryptographic keys.

**Rationale**: Physical security requires fundamentally different controls (tamper-evident hardware, secure enclaves, hardware security modules) that exceed the scope of network-based protections.

#### Cryptographic Implementation Trust
The platform assumes correct implementation of cryptographic primitives in underlying operating systems and hardware. **Out of scope**: Implementation flaws in AES, SHA, elliptic curve cryptography, and random number generation.

**Rationale**: Cryptographic implementation security requires specialized analysis and formal verification beyond the scope of network architecture research.

#### Supply Chain Integrity
Initial device provisioning assumes trustworthy hardware and firmware. **Out of scope**: Malicious firmware implanted during manufacturing, hardware backdoors in silicon, compromised software supply chains.

**Rationale**: Supply chain security requires specialized controls (hardware attestation, verified boot chains, trusted platform modules) that represent distinct research areas with different threat models and defensive approaches.

#### Scalability Boundaries
The current architecture targets small-to-medium deployments (100-1000 devices). **Out of scope**: Enterprise-scale deployments requiring distributed management planes, hierarchical authentication systems, or specialized high-throughput hardware.

**Future work**: Extension to larger deployments through management plane federation and hardware acceleration of cryptographic operations.