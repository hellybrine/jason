# Jason: Modular Edge Security Fabric

Jason is an experimental project exploring how to bring enterprise-level cybersecurity to the edge using affordable, commodity hardware. Built using widely-available single-board computers, it implements a layered, distributed network security architecture inspired by zero-trust principles.

## Motivation

With the rapid growth of IoT and edge computing, securing increasingly complex and diverse devices is a significant challenge. Current solutions tend to be either expensive enterprise offerings (₹8,30,000+) or academic prototypes with limited practical guidance.

I'm working to create a practical, transparent, and extensible platform that demonstrates how rigorous security engineering can be applied effectively on modest hardware for under ₹16,600.

## Core Features

- Clear architectural separation across three hardware nodes handling access control, network segmentation, and threat intelligence
- Strong wireless security with WPA3-SAE and centralized authentication via RADIUS
- Hardware-enforced network segmentation using OpenWrt and IPFire
- Integrated detection and deception with crowd-sourced threat intelligence and honeypots
- Containerized security services on Raspberry Pi for scalability and modularity
- Fully documented and version-controlled setup supporting reproducible research and experimentation

## Use Cases

- Small to medium IoT networks requiring strong segmentation and access control
- Applied network security research, including testing new detection and response methods
- Educational environments for hands-on security learning and experimentation
- Proof-of-concept deployments for cost-effective, high-assurance security architectures

## Architecture

The system is composed of three distinct planes:

1. **Access Plane (NanoPi R1S)**: Manages secure wireless onboarding and device authentication
2. **Segmentation Plane (NanoPi R2S)**: Enforces network zones and isolation through VLANs and firewall policies  
3. **Services Plane (Raspberry Pi 4)**: Provides centralized intelligence including DNS resolution, sinkhole services, threat detection, and deception

## Getting Started

To build your own deployment, you'll need:

- NanoPi R1S for the access plane
- NanoPi R2S for the segmentation plane
- Raspberry Pi 4 for the services plane
- Class 10 microSD cards (32GB minimum)
- Basic networking equipment

Detailed setup instructions, configuration guides, and deployment procedures are available in the documentation.

## Current Status

This is an active research project with ongoing development. While functional, there are areas I'm continuing to improve:

- Performance optimization for higher-throughput scenarios
- Enhanced hardware security integration (TPM, secure boot)
- Simplified deployment and management tools
- Extended monitoring and alerting capabilities

## Contributing

I welcome feedback, suggestions, and contributions from the community. Whether you're interested in:

- Testing deployments and sharing experiences
- Improving documentation or setup procedures
- Adding new security features or detection methods
- Using Jason as a research platform

Your input helps make this project more robust and useful.

## License

Jason is released under the MIT License. See LICENSE for details.

***

*Built with curiosity and a commitment to making practical security accessible.*