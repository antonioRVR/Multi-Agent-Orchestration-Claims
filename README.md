Automated Insurance Claims Processing: Multi-Agent Orchestration on Azure

## Prerequisites

- Azure subscription with Contributor access
- Python 3.10+
- Azure CLI (`az`) installed and authenticated (`az login`)
- A terminal (bash, PowerShell, or WSL)

##  📌 Overview

This project implements a production-ready, multi-agent AI pipeline designed to automate the triage and decision-making process for insurance claims. Built on Azure AI Foundry and orchestrated via Python, the system evaluates incoming claims for completeness, assesses fraud risk, and provides actionable business decisions (Approve, Request Documents, or Escalate to SIU).

The development environment was entirely containerized and cloud-native, utilizing GitHub Codespaces to ensure a reproducible, infrastructure-agnostic workflow.



🏗️ Architecture & Workflow

The pipeline utilizes a Directed Acyclic Graph (DAG) approach, orchestrating two specialized LLM agents:

Claims Triage Agent: Ingests raw claim data (documents, metrics, policy coverage). It validates completeness thresholds and analyzes fraud risk scores.

Claims Decision Agent: Consumes the structured output from the Triage Agent. It applies business logic to recommend the final action based on urgency and risk flags.

Both agents were developed using the Azure AI SDK and subsequently deployed as persistent cloud endpoints via Azure AI Foundry Workflows.

⚙️ Technical Stack

Cloud Infrastructure: Azure AI Foundry, Azure Cognitive Services.

Orchestration: Python (SDK), Azure AI Workflows.

Models Used: gpt-5-mini (optimized for high-throughput, low-latency triage).

Environment: GitHub Codespaces (Dockerized Cloud Dev Environment).

Monitoring & Evaluation: Azure App Insights, Azure AI LLM-as-a-judge evaluators.

🚀 Key Technical Challenges Overcome

During the transition from local experimentation to a cloud-production environment, several architectural challenges were addressed:

1. LLM-as-a-judge Evaluation Pipeline Bottlenecks

The Problem: Automated evaluation runs (measuring Fluency and Coherence) failed with (UserError) Response string cannot be empty. The agents were emitting raw Tool Calls without conversational text, causing the evaluation SDK to crash due to a lack of parseable string output.
The Solution: Engineered a strict prompt-injection protocol within agents.py. The agents were forced to emit a conversational state update (e.g., "Acknowledged. I am launching the assessment tool for claim CLM-XXX now...") synchronously with the tool call. This provided the necessary string output for the Azure evaluators, achieving a 100% success rate across all test iterations.

2. State Management & Context Hallucinations

The Problem: The Triage Agent exhibited poor coherence scores (2.0) by ignoring provided metrics and asking the user for inputs already present in the prompt, occasionally hallucinating default fallback IDs (CLM-001).
The Solution: Refactored the System Prompt architecture to enforce dynamic extraction. The agent was restricted from asking clarifying questions when data was present and forced to inject the dynamically parsed Claim ID into its operational output, drastically improving contextual coherence.

3. Cloud vs. Local Tool Execution

The Problem: When migrating the Python orchestrated workflow to a visual Azure Foundry Workflow, the nodes stalled (0 tokens in/out). The cloud-hosted agents were attempting to call local Python functions (assess_claim) that did not exist in the cloud environment.
The Solution: Decoupled the physical tool dependencies from the cloud workflow. By unbinding the specific tools in the Azure Portal and embedding the structured context directly into the pipeline payload, the agents successfully processed the data purely via LLM reasoning, enabling seamless visual orchestration and accurate tracing.

📈 Business Impact

Latency Reduction: Automates the initial assessment of claims, allowing human adjusters to focus only on cases flagged as "Warning" or "Critical".

Fraud Mitigation: Deterministic flagging of claims where the damage estimate drastically deviates from physical evidence or where completeness is below 80%.

👨‍💻 Author

Developed as a showcase of Data Engineering, AI Orchestration, and Cloud Deployment skills. Focused on pragmatic, scalable, and cost-efficient backend solutions.



## Getting Started

1. Clone this repo and pick a scenario folder (`factory/`, `claims/`, or `callcenter/`)
2. Start with **Challenge 0** — it provisions everything you need
3. Work through challenges 1–4 in order; each builds on the previous one
4. The `agents.py` and `deploy.py` scripts are ready to run — read the README in each challenge folder for what to do