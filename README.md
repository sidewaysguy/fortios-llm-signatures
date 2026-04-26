# FortiOS Local LLM Signature Library

> Custom F-SBID application control signatures for identifying local and network-hosted Large Language Model infrastructure on FortiOS.

[![FortiOS](https://img.shields.io/badge/FortiOS-7.6.6-red?style=flat-square)](https://docs.fortinet.com/product/fortigate/7.6)
[![Signatures](https://img.shields.io/badge/Signatures-35-blue?style=flat-square)](#signature-index)
[![Category](https://img.shields.io/badge/Category-36%20GenAI-green?style=flat-square)](#requirements)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

**[📖 View the Interactive Reference →](https://sidewaysguy.github.io/fortios-llm-signatures/docs/signature-reference.html)**

---

## Overview

FortiGuard's native application signatures cover cloud AI services such as ChatGPT, Claude, Gemini, and Ollama (cloud traffic). This library fills the gap for **local and internal LLM infrastructure** — the same API patterns running inside your network where there is no cloud destination for FortiGuard to match on.

Use these signatures to:
- **Identify** which AI inference servers are running on your network (LM Studio, Ollama, llama.cpp, vLLM, etc.)
- **Detect** which model families and fine-tune variants are being used
- **Identify** which client tools are making requests (AnythingLLM, Claude Code, Python scripts, etc.)
- **Enforce** model-level allow/deny policy through Application Control profiles
- **Flag** high-risk model variants such as uncensored fine-tunes that bypass AI content guardrails

### Relationship to FortiGuard Signatures

These signatures **complement** — they do not replace — FortiGuard signatures. FortiGuard handles cloud-destined traffic. This library handles the same OpenAI-compatible and Anthropic-compatible API formats when they appear on internal network traffic.

---

## Tested Environment

| Component | Version |
|-----------|---------|
| FortiOS | 7.6.6 |
| FortiGate hardware | FSFADOM3 |
| LM Studio | 0.4.x |
| AnythingLLM | Current (April 2026) |
| SSL Inspection | Deep inspection profile |

> **Compatibility note:** These signatures use standard F-SBID syntax and `config application custom`, which has been available since FortiOS 6.x. They should work on FortiOS 7.4+ however they have only been tested and confirmed working on 7.6.6. Category 36 (GenAI) is required and was introduced in FortiOS 7.4. If you are running an earlier version, update the category to the most appropriate available option.

---

## Requirements

- FortiOS 7.4 or later (for Category 36 GenAI)
- Application Control license / FortiGuard subscription
- **Deep Packet Inspection** (SSL inspection profile) applied to the firewall policy for HTTPS traffic
- Application Control profile applied to relevant firewall policies

---

## Signature Library — 35 Signatures

### Infrastructure & Client Identification (01–08)

Matches on URI path and User-Agent header. These fire regardless of which model is loaded.

| # | Signature Name | Matches On | What It Detects |
|---|----------------|-----------|-----------------|
| 01 | `LM.Studio.Native.API` | URI `/api/v` | LM Studio native REST API |
| 02 | `AnythingLLM.API` | URI `/api/v1/workspace` | AnythingLLM server |
| 03 | `Local.LLM.OpenAI.Compat` | URI `/v1/chat/completions` | Any OpenAI-compatible local server |
| 04 | `AnythingLLM.OpenAI.SDK` | Header `OpenAI/JS` | AnythingLLM client via JS SDK |
| 05 | `Local.LLM.Anthropic.Compat` | URI `/v1/messages` | Any Anthropic-compatible local server |
| 06 | `LM.Studio.Anthropic.API` | URI + Header | LM Studio Anthropic endpoint (Claude Code path) |
| 07 | `Client.ClaudeCode` | Header `claude-cli` | Claude Code CLI — local or cloud |
| 08 | `Client.OpenAI.Python.SDK` | Header `AsyncOpenAI/Python` | Python openai SDK scripted access |

### Base Model Families (09–26)

Matches on `/v1/chat/completions` URI + model name in the HTTP POST body. Requires DPI.

| # | Signature Name | Pattern | Creator |
|---|----------------|---------|---------|
| 09 | `Model.Llama` | `llama` | Meta |
| 10 | `Model.Mistral` | `mistral` | Mistral AI (covers Mixtral, Ministral, Magistral, Devstral) |
| 11 | `Model.Phi` | `phi` | Microsoft |
| 12 | `Model.Gemma` | `gemma` | Google |
| 13 | `Model.Qwen` | `qwen` | Alibaba |
| 14 | `Model.DeepSeek` | `deepseek` | DeepSeek AI (weight 65 — see priority notes) |
| 15 | `Model.Nemotron` | `nemotron` | NVIDIA |
| 16 | `Model.LFM` | `lfm` | Liquid AI |
| 17 | `Model.GLM` | `glm` | Z.ai / Zhipu |
| 18 | `Model.Granite` | `granite` | IBM |
| 19 | `Model.GPT-OSS` | `gpt-oss` | OpenAI (open source models) |
| 20 | `Model.OLMo` | `olmo` | Allen AI |
| 21 | `Model.Ernie` | `ernie` | Baidu |
| 22 | `Model.MiniMax` | `minimax` | MiniMax |
| 23 | `Model.Falcon` | `falcon` | TII |
| 24 | `Model.Command` | `command` | Cohere (monitor for false positives) |
| 25 | `Model.InternLM` | `internlm` | Shanghai AI Lab |
| 26 | `Model.Solar` | `solar` | Upstage |

### Fine-Tune Organizations (27–35)

Matches fine-tuned model variants by organization. Weight 62 ensures these outrank base model signatures — a `nous-hermes-2-llama-3.1-8b` will correctly identify as `Model.Hermes`, not `Model.Llama`.

| # | Signature Name | Pattern | Organization | Notes |
|---|----------------|---------|-------------|-------|
| 27 | `Model.Hermes` | `hermes` | NousResearch | Hermes 2/3/4, OpenHermes |
| 28 | `Model.Dolphin` | `dolphin` | Eric Hartford | ⚠️ **UNCENSORED** — see security note |
| 29 | `Model.Zephyr` | `zephyr` | HuggingFace H4 | DPO fine-tunes |
| 30 | `Model.OpenChat` | `openchat` | OpenChat Project | |
| 31 | `Model.Wizard` | `wizard` | Microsoft Research | WizardLM, WizardCoder, WizardMath |
| 32 | `Model.Vicuna` | `vicuna` | UC Berkeley LMSYS | |
| 33 | `Model.Orca` | `orca` | Microsoft + Community | Orca-2, OpenOrca |
| 34 | `Model.Airoboros` | `airoboros` | jondurbin | Creative/roleplay focus |
| 35 | `Model.Phind` | `phind` | Phind | Code-specialized |

---

## Weight Hierarchy

When multiple signatures match the same session, FortiOS selects the highest weight. The hierarchy is intentional:

```
68  Model.Dolphin     — uncensored, always identified first
65  Model.DeepSeek    — wins over Llama for distilled model variants
62  Fine-tuner orgs   — win over base model signatures
60  Base model families
55  Client signatures (header-based)
50  App/server signatures (endpoint-based)
30  OpenAI-compat catch-all
25  Anthropic-compat catch-all
```

---

## ⚠️ Security Note — Model.Dolphin

The Dolphin model series by Eric Hartford is explicitly designed and marketed as **uncensored** — all built-in AI content safety refusals are removed. Users running Dolphin locally bypass AI content guardrails entirely, regardless of what policies are applied at the AI service level.

`Model.Dolphin` is assigned weight 68 (highest in this library) to ensure it is always correctly identified. It is **strongly recommended** to promote this signature to `action block` as the first enforcement step once the monitor phase confirms correct matching.

---

## Quick Start

**1. Paste signatures into the FortiOS CLI:**

For the complete set in one paste, use `signatures/00-all-signatures.conf`.

For selective deployment:
```
signatures/01-infrastructure.conf    # Server endpoints + client identification
signatures/02-base-models.conf       # Base model families
signatures/03-fine-tuners.conf       # Fine-tune organizations
```

**2. Verify signatures saved correctly:**
```
show application custom
```

**3. Note the auto-assigned appids:**
```
show application custom | grep "edit\|appid"
```

**4. Add to your Application Control profile:**
```
config application list
  edit "your-profile-name"
    config entries
      edit 0
        set application <appid>
        set action monitor
      next
    end
  next
end
```

**5. Apply the profile to your firewall policy with a deep inspection SSL profile.**

**6. Review logs before enforcing:**
```
Log & Report > Security Events > Application Control
```
Start with `action monitor`. Once matching is confirmed, promote individual signatures to `action block` as appropriate for your environment.

---

## Important Technical Notes

### Body Matching and TCP Reassembly

The IPS engine does not reassemble HTTP body data across TCP packets. The `"model"` field appears near the start of every OpenAI-compatible POST body and is reliably captured in the first packet for normal API requests. However, sessions with very large pre-populated conversation histories could push the model field into a later packet. Use packet capture to verify matching if issues arise.

### Claude Code Uses Anthropic Format, Not OpenAI Format

Claude Code sends requests to `/v1/messages`, not `/v1/chat/completions`. The per-model body signatures (09–35) only match on `/v1/chat/completions`. Use `Client.ClaudeCode` (#07) and `LM.Studio.Anthropic.API` (#06) to detect Claude Code traffic. Model-level identification of Claude Code sessions requires packet capture.

### HTTP and HTTPS Coverage

All signatures use `--service HTTP` which covers both HTTP and HTTPS traffic. HTTPS coverage requires a deep inspection SSL profile on the firewall policy — without it, only HTTP traffic is inspected. No syntax changes to the signatures are required.

### Ports

These signatures do not rely on port matching. LM Studio defaults to port 1234, AnythingLLM to port 3001, but both are user-configurable. Port-independent URI and header matching ensures signatures fire regardless of the port in use.

---

## Reference Documentation

The full interactive reference with navigation, weight hierarchy, priority resolution, coverage matrix, and FortiGuard integration notes is available online:

**[📖 Interactive Reference →](https://sidewaysguy.github.io/fortios-llm-signatures/docs/signature-reference.html)**

Or open locally from the repo:
```
docs/signature-reference.html
```

---

## Repository Structure

```
fortios-llm-signatures/
├── README.md
├── CHANGELOG.md
├── CONTRIBUTING.md
├── LICENSE
├── signatures/
│   ├── 00-all-signatures.conf      # Complete set — single paste
│   ├── 01-infrastructure.conf      # Sigs 01–08
│   ├── 02-base-models.conf         # Sigs 09–26
│   └── 03-fine-tuners.conf         # Sigs 27–35
└── docs/
    └── signature-reference.html    # Full interactive reference
```

---

## Contributing

Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md) for the full process, signature standards, and submission requirements.

Quick summary: fork the repo, make your changes on a branch, open a Pull Request. All PRs require review before merging to `main`. Use the Issue templates for signature requests, false positive reports, and compatibility reports.

---

## Disclaimer

These signatures are provided as-is for network visibility and policy enforcement purposes. They are tested on FortiOS 7.6.6 and may require adjustment for other versions. Always deploy in monitor mode first and validate matching against your specific traffic before enabling blocking. This is not an official Fortinet or Anthropic product.

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.
