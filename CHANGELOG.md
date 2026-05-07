# Changelog

All notable changes to the FortiOS Local LLM Signature Library.

Format: `[version] YYYY-MM-DD — description`

---

## [1.2.0] 2026-05-06

### Added — AnythingLLM.NativeAPI signature (sig 04b)

Traffic analysis confirmed that AnythingLLM makes two types of requests to LM Studio:

1. **Model enumeration** — GET `/api/v0/models` and `/api/v1/models` using `user-agent: node` and `sec-fetch-mode: cors`
2. **Chat completions** — POST `/v1/chat/completions` using `user-agent: OpenAI/JS 4.95.1`

The existing `LM.Studio.Native.APIv1` signature matched AnythingLLM's model enumeration calls because both LM Studio polling and AnythingLLM's backend use `user-agent: node`. The differentiator is `sec-fetch-mode: cors`: AnythingLLM's backend uses the Node.js 18+ native `fetch()` API which adds this header; LM Studio's own polling uses the Node.js `http` module which does not.

**New signature:** `AnythingLLM.NativeAPI` uses a triple-condition match:
- `/api/v` in URI (covers both v0 and v1 native LM Studio endpoints)
- `node` in header (excludes browser-based traffic to cloud AI services using `/api/v1/`)
- `sec-fetch-mode` in header (distinguishes Node.js fetch from http module)

**Weight 53** — wins over `LM.Studio.Native.APIv1` (weight 50) when AnythingLLM is the client; does not compete with client identification signatures (weight 55). `LM.Studio.Native.APIv1` still correctly fires at weight 50 for LM Studio's own background polling (no `sec-fetch-mode`).

This means a complete AnythingLLM session now produces an identifiable chain across flows:
- Model enumeration → `AnythingLLM.NativeAPI` (this sig)
- Chat POST with known model → model signature wins (e.g., `Model.Qwen`)
- Chat POST with unknown model → `AnythingLLM.OpenAI.SDK` wins

**Total signature count: 38** (was 37)

**Tested on FortiOS 7.6.6** — AnythingLLM connecting to LM Studio Qwen model, confirmed 2026-05-06.

### Fixed — Model.Solar false positive

Production logs confirmed `Model.Solar` firing simultaneously with `Model.Qwen` on the same session when AnythingLLM was used. The word "solar" appeared in POST body message content (e.g., a conversation about solar energy), not in the `"model"` field. Same class of body-text false positive as the `Model.Command` fix in v1.1.0.

**Fix:** Pattern changed from `solar` to `solar-`. All Upstage Solar model names use a hyphenated form (`solar-pro`, `solar-mini`, `solar-10.7b-instruct-v1.0`). The hyphenated string is highly unlikely to appear in natural language chat content where "solar" alone is common ("solar panels", "solar system", "solar energy").

**Tested on FortiOS 7.6.6** — false positive reproduced with `solar` pattern, confirmed absent with `solar-` pattern, 2026-05-06.

### Fixed — Proactive hyphen anchoring for 6 base model signatures

Body-text false positives are a structural risk for any model family whose name is also a common English word. Following the same fix pattern as `Model.Command` (v1.1.0) and `Model.Solar` (above), six additional signatures were updated to require a trailing hyphen — anchoring the match to the model-name field format (`modelname-version-size`) rather than free-form word occurrence.

| Signature | Old pattern | New pattern | Why safe |
|-----------|-------------|-------------|----------|
| `Model.Llama` | `llama` | `llama-` | All Meta Llama 2/3/4 deployments use `Llama-N.x-...` |
| `Model.Phi` | `phi` | `phi-` | All Microsoft Phi models use `Phi-3-...`, `Phi-4`, `Phi-4-mini` |
| `Model.Gemma` | `gemma` | `gemma-` | All Google Gemma models use `gemma-N-...` |
| `Model.Granite` | `granite` | `granite-` | All IBM Granite models use `granite-N.x-...` |
| `Model.Falcon` | `falcon` | `falcon-` | All TII Falcon models use `falcon-Nx` |
| `Model.Ernie` | `ernie` | `ernie-` | All Baidu Ernie models use `ernie-N.x` |

`Model.Mistral` was intentionally **not** changed: Mixtral, Magistral, and Devstral are matched via the `mistralai` publisher prefix in the model ID string. The string `mistral-` does not appear in `mistralai/Mixtral-...`, so adding the hyphen would silently drop all Mixtral-family coverage. The word "mistral" is rare enough in natural conversation that the false positive risk is acceptable.

---

## [1.1.0] 2026-05-06

### Added — Client.CherryStudio signature

Traffic analysis comparing AnythingLLM and Cherry Studio clients connecting to the same Qwen model hosted on LM Studio confirmed Cherry Studio has a highly distinctive Electron-based User-Agent:

```
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) CherryStudio/1.9.4 Chrome/146.0.7680.188 Electron/41.2.1 Safari/537.36
```

Without a dedicated signature, Cherry Studio was identified as:
- `HTTP.BROWSER_Chrome` (Web.Client category) for `/v1/models` calls — incorrect category
- `Local.LLM.OpenAI.Compat` (catch-all) for `/v1/chat/completions` calls — not client-identified

The new `Client.CherryStudio` signature matches the `CherryStudio` token in the User-Agent header at weight 55, consistent with other client identification signatures. This correctly identifies Cherry Studio sessions across all URI paths regardless of which inference server it connects to.

**Total signature count: 37** (was 36)

**Tested on FortiOS 7.6.6** — CherryStudio/1.9.4 connecting to LM Studio Qwen model, confirmed 2026-05-04.

### Fixed — Model.Command false positive confirmed and resolved

Log analysis confirmed that `Model.Command` fires simultaneously with `Model.Qwen` during AnythingLLM sessions (both at 15:53:09 and 16:20:43). The Qwen model being queried does not have "command" in its model name — the pattern was matching the word "command" appearing in the **prompt or message content** rather than in the `"model"` field of the POST body.

**Fix:** Pattern changed from `command` to `command-r`. All Cohere Command models available for local inference use this string in their names (`command-r`, `command-r-plus`, `command-r7b`). The hyphenated form is highly unlikely to appear in natural language chat content, making it a reliable discriminator.

If you deployed `Model.Command` from a prior release, re-apply the updated signature from `signatures/02-base-models.conf` or `signatures/00-all-signatures.conf`.

**Tested on FortiOS 7.6.6** — false positive reproduced with old pattern, confirmed absent with `command-r` pattern, 2026-05-06.

### Fixed — AnythingLLM.OpenAI.SDK weight corrected to 55

`AnythingLLM.OpenAI.SDK` was incorrectly set to weight 60 — the same weight as base model signatures. When both signatures matched the same flow (e.g., AnythingLLM sending a POST to `/v1/chat/completions` with a recognised model), FortiOS logged the model signature and suppressed the client signature, making AnythingLLM invisible in the application control logs whenever a model was identified.

**Fix:** Weight lowered from 60 to 55, consistent with all other client identification signatures (`Client.ClaudeCode`, `Client.CherryStudio` — both weight 55). The intended behaviour is now correct: model signatures (60) win for identified-model sessions; `AnythingLLM.OpenAI.SDK` appears only when no model signature fires (unrecognized model), and wins over the OpenAI-compat catch-all (weight 30).

If you deployed `AnythingLLM.OpenAI.SDK` from a prior release, re-apply the updated signature from `signatures/01-infrastructure.conf` or `signatures/00-all-signatures.conf`.

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
