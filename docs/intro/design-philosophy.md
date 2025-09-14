# Design Philosophy

Traditional network security operates on the **"castle and moat" fallacy**: establish a hard perimeter, then trust everything inside. This approach catastrophically fails in IoT environments, where devices arrive with unknown firmware, undocumented network behaviors, and manufacturer backdoors that persist for device lifetimes.

Jason's design philosophy emerges from a simple recognition: **if you cannot secure the endpoints, you must secure the spaces between them.** This drives five core principles:

## Core Principles

**1. Deny-by-Default as Digital Architecture**  
Every network flow requires explicit authorization. This isn't just policy—it's **structural enforcement** where unauthorized communication is architecturally impossible, not merely forbidden. Traditional "allow-by-default" networks create attack surfaces that scale with device count; deny-by-default networks create attack surfaces that scale with *authorized services*.

**2. Identity-Driven Access Control**  
Network privileges attach to cryptographically verified device identity (MAC address validation, 802.1X certificates) rather than ephemeral IP assignments. A compromised device cannot simply change addresses to escape its security context. This **binding of identity to privilege** prevents the address-hopping techniques common in IoT malware.

**3. Failure Domain Isolation**  
System functions distribute across independent single-board computers to minimize blast radius. Edge access controls, firewall enforcement, and service hosting operate on separate hardware. When one component fails—whether through attack, misconfiguration, or hardware failure—**the security model degrades gracefully** rather than collapsing entirely.

**4. Intelligence-Driven Defense**  
Passive monitoring generates insufficient signal in IoT environments where normal and malicious traffic often appear identical. Jason implements **active defense mechanisms**: DNS response policy zones block known-bad domains, honeypots convert reconnaissance into detection events, and community-driven behavioral analysis (CrowdSec) enables adaptive response to emerging threats.

**5. Reproducible Security Research**  
All configurations exist as version-controlled artifacts—Docker compositions, IPFire zone definitions, OpenWrt UCI configs. This enables **systematic security evaluation**: researchers can reproduce exact network conditions, modify specific policy parameters, and measure security outcomes quantitatively.

## Tradeoffs

**Cost vs. Assurance**  
Single-board computers provide adequate computational resources for cryptographic operations and deep packet inspection while remaining economically viable for deployments. Jason demonstrates that **software-defined security architectures** can achieve enterprise-grade protection (to a degree) without enterprise-grade hardware costs.

**Simplicity vs. Feature Completeness**  
The platform prioritizes mature, well-understood open-source components (OpenWrt, IPFire, Unbound, CrowdSec) over feature-rich commercial alternatives. This **transparency-for-functionality** tradeoff enables security researchers to audit, modify, and extend the platform without reverse-engineering proprietary implementations.

**Local-First Operation**  
Critical security enforcement occurs entirely on-premises. External services provide supplementary threat intelligence and community detection signatures, but **network security never depends on cloud connectivity**. This architectural choice prevents cloud service outages from degrading local security posture and eliminates cloud providers as potential attack vectors.

These principles collectively address the fundamental challenge of IoT security: **how to maintain network security when you cannot trust the devices on your network.**