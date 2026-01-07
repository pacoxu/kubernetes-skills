# Kubernetes Skills

A collection of Kubernetes skills and knowledge for Claude AI assistant.

## Overview

This project provides structured Kubernetes knowledge organized into skill modules that can be used by Claude to better understand and work with Kubernetes concepts, commands, and best practices.

## Skills

- **kubernetes-basics**: Core Kubernetes concepts including Pods, Deployments, Services, and Namespaces
- **kubernetes-deployment**: Deployment strategies, health checks, and scaling
- **kubernetes-networking**: Services, Ingress, Network Policies, and DNS
- **kubernetes-storage**: Volumes, PersistentVolumes, PersistentVolumeClaims, and Storage Classes
- **kubernetes-security**: Service Accounts, RBAC, Secrets, Security Context, and Pod Security Standards

## Structure

```
kubernetes-skills/
├── .claude-plugin/
│   └── manifest.json          # Plugin configuration
├── skills/
│   ├── kubernetes-basics.md
│   ├── kubernetes-deployment.md
│   ├── kubernetes-networking.md
│   ├── kubernetes-storage.md
│   └── kubernetes-security.md
└── README.md
```

## Usage

This project follows the structure of [obsidian-skills](https://github.com/kepano/obsidian-skills) and is designed to be used as a Claude plugin. The skills are organized as markdown files that contain Kubernetes knowledge, examples, and best practices.

## Roadmap

The full roadmap for integrating Claude agents with Kubernetes (Phase 1–3) has been moved to `ROADMAP.md`.  
See that file for detailed milestones, architecture plans, and AI infrastructure integration goals.

## Contributing

Feel free to add more Kubernetes skills or improve existing ones. Each skill file should be well-organized with clear sections, examples, and practical information.

## License

Apache License 2.0
