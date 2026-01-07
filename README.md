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

- **Phase 1: Q1 2026 – Core Integration & MVP**
  - 在 Kubernetes 集群中部署 Claude Agent（例如使用 kagent controller 或自研 Operator），以一个常驻的 “SRE 机器人” 形式接入集群 API，使用严格的 RBAC 权限控制。[参考](https://www.cloudnativedeepdive.com/kagent-claude-k8s-your-private-agentic-troubleshooter/)
  - 安装 / 开发核心 Kubernetes 相关 Claude Skills，例如：Kubernetes manifests skill（帮助生成和校验 YAML）、集群运维 skill（封装常见 `kubectl` 操作、Pod 状态检查、日志查看）等。[参考示例 skill](https://github.com/sairin1202/claude-skill-factory/blob/627c139ca3b488d83a94b09d49f9a3bf155fc7ce/skills/thebushidocollective-han-jutsu-jutsu-kubernetes-skills-kubernetes-manifests-skill-md.json#L6-L14) 和 [Kubernetes Manager skill](https://mcpmarket.com/tools/skills/kubernetes-manager)
  - 通过 ChatOps（如 Slack、CLI 集成）交互式使用 Claude Agent，只读建议模式：例如“帮我生成一个镜像 Y、3 副本的 Deployment”或“帮我看下 Pod X 的日志”，由 Agent 输出 YAML 或命令，由人类审核后执行。
  - 初期能力聚焦在：资源查看（pods/nodes/events）、清单生成（自动加 resource limits、probes、labels 等）、基础故障分析（CrashLoopBackOff、调度失败等，仅给出建议不自动修复）。

- **Phase 2: Q2 2026 – Deep Kubernetes Features**
  - 为高级 Kubernetes 能力设计独立 skills，并补充相应知识文档和示例：
    - **Kueue**：批任务调度与 gang scheduling，支持大模型/批训练作业的队列管理与公平调度。[参考](https://kubernetes.io/blog/2022/10/04/introducing-kueue/)
    - **DRA（Dynamic Resource Allocation）**：面向 GPU 等特殊资源的按需分配与调度（可结合 NVIDIA DRA Driver 等实现）。[参考](https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/)
    - **Gateway API**：比传统 Ingress 更强的 L4/L7 流量治理模型，适配现代网关与服务网格。
    - **GAIE / LLM Infra 集成**：结合云原生 AI 推理框架（如 Solo.io 的分布式推理方案等）将 Claude Agent 融入 AI 工作负载流量路径。[参考](https://www.solo.io/blog/deep-dive-into-llm-d-and-distributed-inference)
  - 为上述高级能力增加专门的排障与建议 skill（例如“帮我分析 Kueue job 为什么一直 Pending”、“帮我检查 GPU DRA 资源是否正确绑定”）。

- **Phase 3: Q3 2026 – 稳定版 & 自动化闭环**
  - 在严格的策略和审批机制下，逐步开放部分自动执行能力：
    - 自动扩缩容（HPA/KPA 等策略调优建议 + 受控自动执行）。
    - 自动调优（资源 Requests/Limits 调优、探针阈值调优、队列/优先级配置建议）。
    - 部分自动修复 playbook（如：重建异常 Pod、重启无响应的 Deployment、副本数回滚等），并提供“预演 + 执行”两阶段模式。
  - 为企业级场景补充：
    - 审计：记录每一次由 Claude Agent 建议或执行的操作，便于追踪与合规。
    - 回滚：为高风险变更自动生成回滚计划和命令。
    - 策略控制：通过 RBAC、Pod Security、NetworkPolicy 等限制 Agent 可见与可改动的资源范围。

## Contributing

Feel free to add more Kubernetes skills or improve existing ones. Each skill file should be well-organized with clear sections, examples, and practical information.

## License

Apache License 2.0
