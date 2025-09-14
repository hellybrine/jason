# Architecture Overview

Jason implements a **layered edge security fabric** that addresses a fundamental architectural flaw in contemporary IoT security: the conflation of network access provisioning with security policy enforcement. Rather than attempting to secure IoT devices directly, Jason assumes device compromise and focuses on **containment through architectural separation**.

The platform distributes conflicting security responsibilities across three specialized single-board computers, eliminating single points of failure while enabling independent security function evolution. This approach mirrors enterprise security principles, i.e. *least privilege, defense-in-depth, and compartmentalization* but demonstrates their viability on commodity hardware.

## Three-Plane Security Architecture

### Access Plane: Cryptographic Identity Establishment
**Hardware**: NanoPi R1S running OpenWrt  
**Core function**: Convert wireless proximity into authenticated network identity

The access plane resolves the authentication bootstrapping paradox—devices need network access to prove their identity, yet should not receive access until identity is verified. It implements WPA3 authentication with per-station isolation, ensuring that network privileges require cryptographic proof rather than physical proximity.

### Segmentation Plane: Policy as Immutable Law  
**Hardware**: NanoPi R2S running IPFire  
**Core function**: Transform security policy into mathematically enforceable network behavior

The segmentation plane implements zone-based architecture (GREEN/BLUE/ORANGE/RED) with default-deny policies compiled directly to kernel enforcement. Communication between zones requires explicit justification—if a connection cannot be justified by legitimate functionality, it is architecturally impossible rather than administratively discouraged.

### Services Plane: Intelligence Through Controlled Deception
**Hardware**: Raspberry Pi running containerized security services  
**Core function**: Convert attacker reconnaissance into defender intelligence

The services plane implements active defense through cryptographically validated DNS (Unbound + DNSSEC), threat intelligence integration (RPZ), distributed honeypots, and community-driven behavioral analysis (CrowdSec + ClamAV). Rather than reactive monitoring, it creates controlled environments where attacker behavior becomes distinguishable from legitimate device activity.

## Design Rationale

**Hardware Separation**: Each plane addresses incompatible security requirements that cannot be safely co-located. Functional separation ensures graceful degradation rather than catastrophic failure when individual components are compromised.

**Zero-Trust Through Architecture**: Security controls are enforced through technical constraints rather than administrative policies, eliminating the compliance gap between policy specification and implementation.

**Research Platform**: The distributed architecture enables systematic evaluation of IoT security approaches through controlled experimentation while remaining economically accessible.