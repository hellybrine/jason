# NanoPi R1S (OpenWrt): Access Control

The NanoPi R1S implements Jason's **access plane**—the first and most critical security boundary that every IoT device must cross. This component confronts the fundamental bootstrapping problem in wireless security: **how to safely integrate unknown devices into a network without creating attack opportunities during the integration process**.

## Security Challenge: The Authentication Paradox

Traditional wireless security creates a **trust inversion**: devices receive network privileges immediately after authentication, but authentication success provides no information about device intentions, behavioral patterns, or security posture. A compromised smart bulb with valid credentials becomes indistinguishable from a legitimate smart bulb until it begins attacking other devices.

The R1S resolves this paradox by **decoupling authentication from authorization**: proving device identity becomes separate from granting network privileges.

## Technical Implementation

### WPA3-Personal (SAE): Cryptographic Identity Binding
**Security problem solved**: Traditional WPA2-PSK authentication is vulnerable to offline dictionary attacks and does not provide forward secrecy.

WPA3's **Simultaneous Authentication of Equals (SAE)** handshake eliminates these vulnerabilities through:
- **Per-session cryptographic binding** that prevents offline password attacks by incorporating session-specific randomness into the authentication process
- **Forward secrecy guarantees** ensuring that session key compromise cannot decrypt previous communications
- **Management Frame Protection (MFP)** that cryptographically authenticates deauthentication and disassociation frames, preventing spoofed disconnection attacks

The result: **wireless association requires cryptographic proof of authorization**, not merely proximity to the access point.

### RADIUS-Integrated MAC ACL: Identity-Driven Access Control  
**Security problem solved**: MAC address-based filtering is easily bypassed through address spoofing, while DHCP-based access control creates race conditions.

The R1S implements **identity-bound authorization** through RADIUS integration:
- **MAC address validation** occurs immediately after WPA3 authentication, creating a two-factor authentication model (cryptographic credentials + hardware identity)
- **Manual device approval** prevents unauthorized devices from accessing the network even if they possess valid Wi-Fi credentials
- **Persistent identity binding** ensures that network privileges are tied to specific hardware rather than ephemeral IP assignments

This approach transforms device onboarding from an **automatic process into a deliberate security decision**.

### Per-Station Layer-2 Isolation: Attack Surface Elimination
**Security problem solved**: Traditional wireless networks create shared broadcast domains where compromised devices can immediately attack other devices through ARP spoofing, DHCPv6 attacks, and broadcast/multicast abuse.

The R1S implements **microsegmentation at Layer-2**:
- **Individual broadcast domains** for each connected device eliminate lateral communication at the data link layer
- **ARP table isolation** prevents devices from discovering or attacking other devices' network configurations  
- **Forced traffic routing** through higher-layer security controls ensures that all inter-device communication must traverse firewall policy enforcement

The architectural consequence: **device compromise cannot immediately propagate to other devices** because the attack surface for lateral movement is eliminated at the most fundamental network layer.

### Security Hardening: Attack Surface Minimization
**Security problem solved**: Access points typically run unnecessary services and use permissive kernel configurations that create attack opportunities.

The R1S implements **defense in depth** through systematic hardening:
- **Kernel sysctl optimization** applies SYN flood protection, ICMP rate limiting, and ARP validation to prevent denial-of-service attacks against the wireless infrastructure
- **Netfilter rule minimization** permits only essential control traffic (DHCP, DNS, authentication) while blocking unnecessary protocols
- **Service reduction** eliminates non-essential userland processes to minimize the attack surface available to remote attackers

## Research Applications

The R1S enables systematic study of **access control security in resource-constrained environments**:

### IoT Authentication Model Evaluation
The platform provides a controlled environment for evaluating different authentication approaches with devices that lack enterprise security features (802.1X, certificate management, secure key storage).

### Rogue Device Detection Research  
The RADIUS integration enables systematic testing of device identification techniques, behavioral fingerprinting, and anomaly detection during the onboarding process.

### Wireless Attack Surface Analysis
The combination of WPA3, MAC ACL, and Layer-2 isolation creates a testbed for evaluating wireless security controls against realistic IoT attack scenarios.

### Access Layer DoS Resilience Testing
The hardened configuration provides a baseline for measuring the effectiveness of kernel-level protections against various denial-of-service attacks targeting wireless infrastructure.

## Architectural Role

The R1S operates on the principle that **network access is a privilege that must be continuously justified, not a right that is automatically granted**. It transforms wireless connectivity from a binary decision (connected/disconnected) into a **graduated trust establishment process** where device identity, authorization, and behavioral constraints are established before any meaningful network access is granted.

This approach demonstrates that **sophisticated access control is achievable on commodity hardware** without requiring expensive enterprise wireless controllers or cloud-based device management platforms.