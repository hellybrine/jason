# Just Another Secure On‑prem Network

Contemporary IoT deployments exhibit a fundamental architectural flaw: **transitive trust propagation**. When heterogeneous sensor nodes join residential or enterprise networks, they inherit broad network access by default, this creates attack surfaces that extend far beyond the device's intended function. Compromise of a singular IoT endpoints frequently enables lateral movement, data exfiltration, and persistent access to critical network segments.

This is an attempt to addresses such a problem through **architectural enforcement of zero-trust principles** at the network edge. Rather than relying on device-level security controls, which have proven inadequate across ecosystems; Jason implements a deny-by-default fabric that isolates devices while providing controlled, auditable access to necessary services.

Built on commodity single-board computers (NanoPi, Radxa & Raspberry Pi), Jason also demonstrates that **enterprise-grade network security is achievable with consumer hardware** through careful architectural design and open-source tooling.

## High Level Implementation

#### **Microsegmentation Through Policy Compilation**  
Jason implements IPFire's zone-based security model to express network policy as deterministic inter-zone flows. Unlike traditional VLAN-based segmentation, this approach compiles high-level security intent directly to kernel enforcement mechanisms (nftables), eliminating policy-implementation gaps and enabling formal verification of network reachability constraints.

#### **Identity-Anchored Access Control**  
The access plane binds device identity at both L2 and L3 layers through WPA3-Enterprise (802.1X/RADIUS) authentication combined with DHCP fingerprinting and MAC address validation. Per-station isolation prevents lateral movement within broadcast domains, while ephemeral credential rotation limits the impact of credential compromise.

#### **Distributed Defense Services**  
Containerized security services provide layered protection without introducing co-tenancy risks. The architecture includes validating DNS resolution (Unbound with DNSSEC), programmable DNS sinkholes (Response Policy Zones), distributed honeypots for early threat detection, and adaptive behavioral analysis (CrowdSec) integrated with content scanning (ClamAV).

## Platform Design

Jason's configuration is **fully declarative and reproducible**, enabling systematic evaluation of network security policies, IoT threat models, and zero-trust architectures. The platform supports controlled experiments in:

- **Policy effectiveness measurement**: Quantifying the security impact of different segmentation strategies
- **Attack surface analysis**: Systematic evaluation of IoT device attack vectors under various network constraints  
- **Detection algorithm validation**: Testing behavioral and signature-based detection approaches against known IoT malware families
- **Performance characterization**: Measuring the computational and bandwidth overhead of cryptographic authentication and deep packet inspection

## Quick Links
- [Design philosophy](intro/design-philosophy.md)
- [Threat model](intro/threat-model.md)
- [Architecture overview](architecture/overview.md)
- [Get started (setup)](setup/hardware.md)