# Changelog

All notable changes to the FortiOS Local LLM Signature Library.

Format: `[version] YYYY-MM-DD — description`

---

## [1.0.0] 2026-04-26

### Added — Initial release

**Infrastructure signatures (01–08)**
- `LM.Studio.Native.API` — LM Studio native /api/v path
- `AnythingLLM.API` — AnythingLLM /api/v1/workspace path
- `Local.LLM.OpenAI.Compat` — OpenAI-compatible /v1/chat/completions catch-all
- `AnythingLLM.OpenAI.SDK` — AnythingLLM via OpenAI JS SDK User-Agent
- `Local.LLM.Anthropic.Compat` — Anthropic-compatible /v1/messages catch-all
- `LM.Studio.Anthropic.API` — LM Studio Anthropic endpoint (Claude Code path, introduced LM Studio 0.4.1)
- `Client.ClaudeCode` — Claude Code CLI via claude-cli User-Agent
- `Client.OpenAI.Python.SDK` — Python openai SDK scripted access

**Base model signatures (09–26)**
- `Model.Llama` — Meta Llama (weight 60)
- `Model.Mistral` — Mistral AI family including Mixtral, Ministral, Magistral, Devstral (weight 60)
- `Model.Phi` — Microsoft Phi-3, Phi-4 (weight 60)
- `Model.Gemma` — Google Gemma 3, Gemma 4, Gemma 3n (weight 60)
- `Model.Qwen` — Alibaba Qwen3 family (weight 60)
- `Model.DeepSeek` — DeepSeek AI (weight 65 — elevated for distilled model priority)
- `Model.Nemotron` — NVIDIA Nemotron (weight 60)
- `Model.LFM` — Liquid AI LFM2 (weight 60)
- `Model.GLM` — Z.ai GLM series (weight 60)
- `Model.Granite` — IBM Granite 3/4 (weight 60)
- `Model.GPT-OSS` — OpenAI open source models (weight 60)
- `Model.OLMo` — Allen AI OLMo/olmOCR (weight 60)
- `Model.Ernie` — Baidu Ernie (weight 60)
- `Model.MiniMax` — MiniMax M2 (weight 60)
- `Model.Falcon` — TII Falcon (weight 60)
- `Model.Command` — Cohere Command-R (weight 60)
- `Model.InternLM` — Shanghai AI Lab InternLM (weight 60)
- `Model.Solar` — Upstage Solar (weight 60)

**Fine-tuner signatures (27–35)**
- `Model.Hermes` — NousResearch Hermes 2/3/4, OpenHermes (weight 62)
- `Model.Dolphin` — Eric Hartford Dolphin UNCENSORED (weight 68)
- `Model.Zephyr` — HuggingFace H4 Zephyr (weight 62)
- `Model.OpenChat` — OpenChat Project 3.5/3.6 (weight 62)
- `Model.Wizard` — Microsoft Research WizardLM/WizardCoder/WizardMath (weight 62)
- `Model.Vicuna` — UC Berkeley LMSYS Vicuna (weight 62)
- `Model.Orca` — Microsoft Orca-2 and OpenOrca community (weight 62)
- `Model.Airoboros` — jondurbin Airoboros creative/roleplay (weight 62)
- `Model.Phind` — Phind code-specialized CodeLlama (weight 62)

### Tested on
- FortiOS 7.6.6
- LM Studio 0.4.x
- AnythingLLM (current)
- Traffic confirmed matching via FortiOS application control logs

---

## Future — Planned

- Monitor for new model families on lmstudio.ai/models and huggingface.co/models
- Add per-model signatures for additional fine-tune organizations as ecosystem grows
- Consider Ollama-specific URI signatures to complement FortiGuard native Ollama signature
- Review `Model.Command` for false positive rate in production — "command" is a generic term
