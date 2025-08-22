# Transparency Note for CLI Agent for AKS

This transparency note provides information about the capabilities, limitations, and responsible use of the CLI agent for Azure Kubernetes Service (AKS). Understanding these aspects helps you use the system effectively and responsibly.

## What is CLI Agent for AKS?

The CLI Agent for AKS is an AI-powered command-line tool designed to help Azure Kubernetes Service (AKS) users troubleshoot cluster issues efficiently. It analyzes telemetry signals (logs, metrics, events), correlates them across infrastructure and workloads, and provides actionable insights. The agent takes natural language queries as input and returns diagnostic summaries, root cause analyses, and remediation suggestions. The CLI Agent does not include the AI models; users need to provide their own Large Language Model (LLM) API keys for the agent to work.

## What Can CLI Agent for AKS Do?

- Interprets natural language queries
- Executes diagnostic commands
- Returns actionable insights
- Integrates with AKS-native tools and telemetry sources (Kubernetes events, logs, inspector-gadget, Azure and AKS APIs)
- Respects Azure RBAC and identity controls (inherits permissions from Azure CLI, operates in read-only mode by default)
- Allows configuration of AI provider (OpenAI, Azure OpenAI, Anthropic, etc.)
- Outputs include: AI-synthesized summary, root cause analysis, remediation suggestions, diagnostic traces, and tool outputs (kubectl, az, tcpdump)

## Intended Uses

- Human-in-the-loop troubleshooting for AKS clusters
- Read-only interactions with Kubernetes and AKS APIs
- Resource information, health checks, and best practices for AKS

**Not intended for:**
- Generic coding or AI agent use beyond AKS scope
- Internet access for generic questions

**Limitations:**
- May miss subtle signals or misinterpret noisy telemetry
- Requires human validation for suggested mitigations
- Optimized for AKS-specific scenarios

## Responsible Use

- Treat agent output as a starting point, not a final verdict
- Human oversight is essential, especially in complex or high-stakes environments
- Recommended AI models: Azure OpenAI (GPT-4o, GPT-3), OpenAI API, Anthropic, Mistral, etc.

## Evaluation and Metrics

- Internal testing and programmatic evaluations (evals)
- Metrics: Groundedness, UPIA/XPIA Jailbreak, Harmful Content, Conversation Quality
- Bug bashes and red-teaming for common issues (node health, DNS failures, upgrade disruptions, pod scheduling)
- Core metric: accuracy of diagnosis and relevance of recommendations
- User feedback welcomed via feedback form or GitHub issues

## Limitations and Mitigation

### Key Limitations
- **Data access dependency:** Diagnostics depend on user permissions and telemetry availability
- **Token limits:** Large datasets may constrain analysis
- **Limited managed Azure experiences:** Some workflows (e.g., Azure Monitor Alerts) may not be fully supported
- **AI-generated inaccuracies:** Insights are advisory; validate against actual cluster data

### Minimizing Impact
- Ensure diagnostic tools (e.g., Azure Monitor) are properly configured
- Extend capabilities with Azure-mcp or AKS-mcp servers
- Use latest generation reasoning models (GPT-4o, GPT-3, etc.)

## Operational Factors for Responsible Use

- Operates in read-only mode by default
- Write operations require explicit user approval
- Runs locally; supports bring-your-own (BYO) AI providers
- All data processing happens locally for privacy and security
- Configurable verbosity settings (concise summaries or detailed outputs)
- Integrates with Azure identity and RBAC for secure access boundaries

## Feedback Channels

- GitHub issues and PRs on the HolmesGPT repository
- Internal channels during private preview
- Azure support tickets or direct engagement with AKS product team
- Email: aksagentcli@service.microsoft.com

## Plugins in CLI Agent for AKS

Plugins are modular extensions that enhance diagnostic capabilities by integrating external tools, data sources, and domain-specific logic. Three primary types:

### Custom Runbooks
- Predefined, scenario-specific workflows for structured troubleshooting (e.g., DNS failures)

### Toolset Integrations
- Connect to observability platforms (Prometheus, Datadog, Azure Monitor)
- Expose metrics, logs, and alerts for real-time analysis

### MCP Servers
- Model Context Protocol servers expose diagnostic tools and prompt templates
- Enable commands like `kubectl describe`, `az aks show`, deploy debug pods
- Standardize tool invocation and data return for scalability

## Plugin Data and Permissions

- Plugins are pull-only (no outward dataflow except to connected AI models)
- Used to improve diagnostic capabilities via custom runbooks and tool integrations

## Common Issues with Plugins

- Incorrect tool invocation due to prompt logic errors
- Fabricated outputs if plugins return incomplete or ambiguous data
- Configuration errors (e.g., missing telemetry, unsupported cluster configurations)

### Mitigation Strategies
- Verbose logging and error reporting
- Manual override/disable for problematic plugins
- Clear documentation and community support
- Use latest generation LLMs to reduce hallucinations
