# Jason: Just Another On-prem Network  

Jason is a small-form-factor edge fabric designed to securely onboard and operate IoT devices **without exposing your LAN or WAN**. Built on Raspberry Pi and NanoPi boards, it applies **zero-trust principles**: default-deny, per-device isolation, and layered defense across access, segmentation, and services.  

---

## Architecture at a Glance  

### Access Layer – *NanoPi R1S with OpenWrt*  
- WPA3-Personal (SAE) with management frame protection  
- RADIUS-backed MAC ACLs + manual device approval  
- Per-station isolation (no device can see another)  
- Hardened kernel/sysctl and minimized userland  

### Segmentation Layer – *NanoPi R2S with IPFire*  
- Zone-based isolation (GREEN = trusted, ORANGE/BLUE = IoT/guest)  
- Default-deny inter-zone policy with explicit allowlists  
- Strict interface binding, NAT/pinholes only if required  
- Stateful filtering + rate limiting to reduce attack surface  

### Services Layer – *Raspberry Pi with containers*  
- **DNS Security**: Unbound resolver (DNSSEC, QNAME minimization), sinkhole for policy enforcement  
- **Deception**: Honeypots (SSH/HTTP/ICS) to detect and absorb scans  
- **Defense**: CrowdSec for behavioral detection, ClamAV for malware hygiene  
- **Visibility**: Logs + events correlated across firewall, DNS, and honeypots  

---

## Why Jason?  
- **Zero-trust by design**: every device is isolated, every policy is explicit.  
- **Commodity hardware**: reproducible on <$100 SBCs.  
- **Active defense**: blocks, deceives, and learns from attacks.  
- **On-prem focus**: built for labs, homelabs, and small deployments where cloud isn’t an option.