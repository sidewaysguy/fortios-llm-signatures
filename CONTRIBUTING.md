# Contributing to FortiOS Local LLM Signature Library

Thank you for your interest in contributing. This library is most useful when it stays current with the fast-moving LLM ecosystem — new model families, fine-tuners, and client tools appear regularly.

---

## How Contributions Work

All contributions go through a **Pull Request** — nobody pushes directly to `main`, including the maintainer. This ensures every change is reviewed before it becomes part of the library.

**The basic workflow:**

1. Fork the repository to your own GitHub account
2. Create a new branch for your change (e.g. `add-model-qwq` or `fix-command-false-positive`)
3. Make your changes
4. Open a Pull Request back to `main` in this repository
5. Describe what you changed and why
6. The maintainer reviews and merges

If you're new to GitHub and this process is unfamiliar, GitHub's documentation on [forking a repository](https://docs.github.com/en/get-started/quickstart/fork-a-repo) is a good starting point.

---

## What Contributions Are Welcome

### New model family signatures
New base model families that have distinct naming patterns in their model identifiers and are commonly deployed locally. Check [lmstudio.ai/models](https://lmstudio.ai/models) and [huggingface.co/models](https://huggingface.co/models) for reference.

### New fine-tune organization signatures
Community fine-tuners with a distinct model naming pattern across their model catalog.

### New infrastructure / client signatures
New local LLM servers or client tools with identifiable URI paths or User-Agent strings.

### False positive reports
If a signature is firing on traffic it shouldn't — especially `Model.Command` which uses a generic pattern — please report it via an Issue so we can investigate.

### Testing on other FortiOS versions
Confirmation (or failure reports) of these signatures on FortiOS versions other than 7.6.6. Include your FortiOS version, hardware platform, and whether signatures saved and matched correctly.

### Documentation improvements
Corrections, clarifications, or additions to the README or HTML reference.

---

## Signature Submission Requirements

When submitting a new signature via Pull Request, your submission must include all of the following. Pull Requests missing these details will be asked to revise before review.

**1. The signature itself** added to the correct `.conf` file in `signatures/` and also to `signatures/00-all-signatures.conf`.

**2. FortiOS version tested** — which version you pasted the signature into and confirmed it saved without error.

**3. Model name examples** — at least two or three real model identifier strings from HuggingFace or the vendor that the pattern is intended to match. For example:
```
qwq-32b-preview
qwq-32b-q4_k_m
```

**4. Confirmation the signature saved** — paste the output of `show application custom "Your.Signature.Name"` to confirm FortiOS accepted it.

**5. Weight justification** — which weight tier your signature uses and why:
- `60` for base model families
- `62` for fine-tune organizations
- `65`+ only if there is a documented overlap with an existing signature that needs resolving

**6. README update** — add your signature to the appropriate table in `README.md`.

---

## Signature Standards

All signatures in this library follow these standards. Please match them exactly.

**Naming convention:** `Category.Name` in dot notation with title case. Examples: `Model.Llama`, `Client.ClaudeCode`, `LM.Studio.Native.API`.

**Required fields** for every signature:
```
set category 36
set technology 2
set behavior 9
set vendor 0
set protocol "TCP"
set comment "..."
```

**Comment field:** Write a plain English description of what the signature detects and any important notes (e.g. overlap with other signatures, false positive risk). This appears in the FortiOS GUI and helps operators understand what they're looking at.

**Pattern case:** Always include `--no_case` for string patterns so matching is case-insensitive. Model names appear in both upper and lower case depending on the client.

**Flow direction:** Use `--flow from_client` for all request-based detection. We are matching outbound API calls, not responses.

---

## What We Will Not Accept

- Signatures based solely on port numbers — ports are user-configurable and unreliable
- Signatures with patterns that are too generic and likely to produce false positives without a strong justification and testing evidence
- Signatures for cloud AI services — FortiGuard covers those natively
- Changes to existing signature weights without a documented overlap scenario and testing evidence
- Modifications to `00-all-signatures.conf` without the corresponding change in the individual `.conf` file

---

## Reporting Issues

Use GitHub Issues for:
- False positive reports
- Signatures that fail to save on a specific FortiOS version
- Incorrect information in the README or reference documentation
- Suggestions for new signatures you don't have the ability to test yourself

Use the appropriate Issue Template when filing — it will prompt you for the information needed to investigate.

---

## Questions

If you're unsure whether a signature is appropriate for this library, open an Issue with the label `question` before spending time writing and testing it. Happy to discuss.
