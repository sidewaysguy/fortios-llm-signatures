# Changelog

All notable changes to the FortiOS Local LLM Signature Library.

Format: `[version] YYYY-MM-DD — description`

---

## [1.0.1] 2026-05-02

### Fixed — False positive on LM.Studio.Native.API

**Problem:** The original `LM.Studio.Native.API` signature used the pattern `/api/v` in the URI. Traffic analysis confirmed this produces false positives on `chat.qwen.ai`, which uses `/api/v1/` and `/api/v2/` paths for its web frontend internal API calls (destined for public IP `47.77.2.206`). Risk assessment identified the same false positive pattern likely exists for other cloud AI web frontends using versioned REST APIs including Kimi/Moonshot and DeepSeek.

**Affected URIs confirmed in logs:**
- `/api/v2/chat/completions` — Qwen chat endpoint
- `/api/v2/users/status` — Qwen user status
- `/api/v1/auths/` — Qwen authentication
- `/api/v2/configs/setting-config` — Qwen config
- `/api/v2/chats/new` — Qwen new chat
- `/api/v2/tts/config` — Qwen text-to-speech config

**Fix:** Split `LM.Studio.Native.API` into two signatures:

- **`LM.Studio.Native.API`** — pattern changed from `/api/v` to `/api/v0/`. The v0 path is unique to LM Studio's legacy native API. No cloud AI service uses v0 versioning. Zero false positive risk.

- **`LM.Studio.Native.APIv1`** (new) — matches `/api/v1/` URI combined with `node` User-Agent. LM Studio's native API always sends `node` as the User-Agent when making v1 path calls. Cloud AI web frontends are accessed via browsers and never send `node`. This combination reliably distinguishes local LM Studio v1 traffic from cloud service frontend requests.

**Total signature count: 36** (was 35)

**Tested on FortiOS 7.6.6** — confirmed `chat.qwen.ai` no longer triggers `LM.Studio.Native.API` with updated signatures.

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
