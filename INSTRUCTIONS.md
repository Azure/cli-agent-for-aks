# Instructions for CLI Agent for AKS

For questions or issues, please reach out to the team at aksagentcli@services.microsoft.com.

To submit feedback, please fill out our [feedback form](aka.ms/aks/cli-agent/feedback).

## Prerequisites

Before installing and using the CLI Agent, make sure you have the following configured:

1. LLM Key
    - Preferred: Azure OpenAI
    - Supported: OpenAI or other LLM providers compatible with OpenAPI specificaitons - openai/openai-openapi: OpenAPI specification for the OpenAI API
    - You must bring your own API key (see **LLM Keys** section for setup instructions).
2. Azure CLI version 2.76 or higher
    - To verify your Azure CLI version, run the following command:
```bash
az version
```
3. AKS Cluster Access
    - Ensure that your Azure CLI subscription and kubeconfig context are set correctly:
```bash
az account set --subscription <subscription-id>
az aks get-credentials --resource-group <rg_name> --name <cluster_name> --overwrite-existing
kubectl cluster-info
```
The output should show the cluster you intend to use.

## LLM Keys

While you can use any model, we highly recommend you to use the newer models such as GPT-5, GPT-4.1 or Claude Sonnet 4.0. In our experience, they perform much better than other models. You will only need to have one LLM API key from the following options.

### Azure OpenAI (Recommended)

1. Set up an Azure Open AI resource by following the [Microsoft documentation](https://learn.microsoft.com/azure/ai-foundry/openai/how-to/create-resource?pivots=web-portal).
2. [Deploy the model](https://learn.microsoft.com/azure/ai-foundry/openai/how-to/create-resource?pivots=web-portal#deploy-a-model) you plan to use in the Azure AI Foundry Portal.
3. Once deployed, note your API base URL, API version, and API key, then set them as environment variables:
 
```bash
export AZURE_API_BASE="https://<your-endpoint>.openai.azure.com/"
export AZURE_API_VERSION="2025-04-01-preview"
export AZURE_API_KEY="<your-api-key>"
```

**NOTE**: It is necessary to specify the model when using Azure Open AI with az aks agent by using the --model parameter.

### OpenAI

If you have an OpenAI API key:

```bash
export OPENAI_API_KEY="<your-openai-api-key>"
```

### Anthropic

```bash
export ANTHROPIC_API_KEY="<your-anthropic-api-key>"
```

### Gemini

```bash
export GEMINI_API_KEY="<your-gemini-api-key>"
```

### Ollama

```bash
export OLLAMA_API_BASE="<your-ollama-api-key>"
```

### Other LLM providers
We also support any OpenAI compatible model, please reach out to us for support.

## Installation

#### Step 1 – Upgrade Azure CLI to 2.76+
Run the following commands to upgrade and verify your Azure CLI version:

```bash
az upgrade
az version
```

Expected output should include: "azure-cli": "2.76.0" or higher

#### Step 2 – Install the AKS Preview Extension

If you have previously installed the aks-preview extension, remove it by running:

```bash
az extension remove --name aks-preview
```

Next, install the Limited Preview AKS extension - this step might take 5-10 minutes to complete:

```bash
az extension add --source https://aksagentbugbash.blob.core.windows.net/extension/aks_preview-18.0.0b27-py2.py3-none-any.whl --yes --debug
```

#### Step 3 – Verify Installation
After installation, verify that the CLI Agent was successfully installed by running:

```bash
az aks agent --help
```

You should see the command info and arguments listed.

## Usage

### Basic Queries

Try the following example prompts to test the agentic functionality:

```bash
az aks agent "How many nodes are in my cluster?" --model=azure/gpt-4o // replace with your deployed model

az aks agent "What is the Kubernetes version on the cluster?" --model=azure/gpt-4o

az aks agent "Why is coredns not working on my cluster?" --model=azure/gpt-4o

az aks agent “Why is my cluster is a failed state?” --model=azure/gpt-4o
```

By default, the experience uses interactive mode where you can continue asking questions with retained context until you want to exit. To quit the experience, type “/exit”.

### Toolsets
The CLI Agent for AKS includes pre-built integrations for popular monitoring and observability tools through toolsets. Some work automatically with Kubernetes, while others require API keys or configuration.

For AKS, there are specific toolsets which help with the troubleshooting experience. These toolsets appear in the output at the start of the experience:

```bash
az aks agent --model=azure/gpt-4o "How many pods are in the kube-system namespace"

✅ Toolset kubernetes/kube-prometheus-stack
✅ Toolset internet
✅ Toolset bash
✅ Toolset runbook
✅ Toolset kubernetes/logs
✅ Toolset kubernetes/core
✅ Toolset kubernetes/live-metrics
✅ Toolset aks/core
✅ Toolset aks/node-health

Using 37 datasources (toolsets). To refresh: use flag `--refresh-toolsets`  
```

Currently, we support adding the MCP server toolset through the config file.

### Skipping Interactive Mode: --no-interactive
To opt-out of the default interactive mode and run the Agent CLI commands one-off, use the "--no-interactive" flag:

```bash
az aks agent "How many pods are in the kube-system namespace" --model=azure/gpt-4o --no-interactive
az aks agent "Why are the pods in Crashloopbackoff in the kube-system namespace" --model=azure/gpt-4o --no-interactive --show-tool-output
```
 
### Parameters and Options
| Argument | Description |
|--|--|
| --api-key | API key to use for the LLM (if not given, uses environment variables AZURE_API_KEY, OPENAI_API_KEY). |
| --config-file | Path to configuration file. Default: /Users/<>/.azure/aksAgent.config |
| --max-steps | Maximum number of steps the LLM can take to investigate the issue. Default: 10. |
| --model | Model to use for the LLM. |
| --name, -n | Name of the managed cluster. |
| --no-echo-request | Disable echoing back the question provided to AKS Agent in the output. |
| --no-interactive | Disable interactive mode. When set, the agent will not prompt for input and will run in batch mode. |
| --refresh-toolsets | Refresh the toolsets status. |
| --resource-group, -g | Name of the resource group. |
| --show-tool-output | Show the output of each tool that was called during the analysis. |

The az aks agent has a set of subcommands which aid the troubleshooting experience. You can access them by typing `/` inside the interactive mode experience.

| Command | Description |
|--|--|
| /exit | Exit interactive mode |
| /help | Show help message with all commands |
| /clear | Clear screen and reset conversation context |
| /tools | Show available toolsets and their status |
| /auto | Toggle auto-display of tool outputs after responses |
| /last | Show all tool outputs from last response |
| /run | Run a bash command and optionally share with LLM |
| /shell | Drop into interactive shell, then optionally share session with LLM |
| /context | Show conversation context size and token count |
| /show | Show specific tool output in scrollable view |

## Troubleshooting

If you run into any issues using the CLI Agent for AKS, verify the following:

- Make sure your API keys and environment variables are set correctly
- Check if you are using the right model with --model (if needed - Azure OpenAI)
- Confirm cluster connectivity using: kubectl cluster-info
- If you see the requests retrying to /chat/completions in the responses, you are throttled by the token per minute(TPM) limits from the LLM. Increase the TPM limit or apply for more quota [here](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/quotas-limits?tabs=REST#request-quota-increases).
- If outputs vary, it may be due to LLM response variability or intermittent MCP server connections.

### Errors

```bash
Error: litellm.BadRequestError: AzureException BadRequestError - Unsupported value: 'temperature' does not support 1E-8 with this model. Only the default (1) value is supported. User: User: litellm.exceptions.BadRequestError: litellm.BadRequestError: AzureException BadRequestError - Unsupported value: 'temperature' does not support 1E-8 with this model. Only the default (1) value is supported. Error: litellm.BadRequestError: AzureException BadRequestError - Unsupported value: 'temperature' does not support 1E-8 with this model. Only the default (1) value i s supported./
```
**Fix**: Set an environment variable for TEMPERATURE to 1:

```bash
export TEMPERATURE=1
```