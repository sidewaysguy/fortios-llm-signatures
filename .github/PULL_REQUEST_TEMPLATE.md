## Pull Request Type

<!-- Check all that apply -->
- [ ] New model family signature
- [ ] New fine-tune organization signature
- [ ] New infrastructure / client signature
- [ ] False positive fix
- [ ] FortiOS version compatibility update
- [ ] Documentation improvement
- [ ] Other (describe below)

---

## Description

<!-- What does this PR add or change, and why? -->

---

## Signature Details

<!-- Complete this section for any new or modified signatures. Skip for documentation-only changes. -->

**Signature name(s):**

**FortiOS version tested:**

**Hardware platform:**

**Model name examples this pattern matches:**
```
example-model-name-1
example-model-name-2
```

**Output of `show application custom "Your.Signature.Name"`:**
```
(paste output here)
```

**Weight used and justification:**

---

## Checklist

<!-- All items must be checked before a PR will be reviewed -->

- [ ] Signature added to the correct individual `.conf` file in `signatures/`
- [ ] Signature added to `signatures/00-all-signatures.conf`
- [ ] README.md updated with new signature in the appropriate table
- [ ] Signature saved without error on FortiOS (confirmed via `show application custom`)
- [ ] Pattern includes `--no_case`
- [ ] Pattern uses `--flow from_client`
- [ ] Comment field describes what the signature detects
- [ ] All required fields set: `category 36`, `technology 2`, `behavior 9`, `vendor 0`, `protocol "TCP"`
- [ ] CHANGELOG.md updated with a brief description of the change

---

## Additional Notes

<!-- Anything else the reviewer should know — overlap with existing signatures, false positive risk, tested traffic scenarios, etc. -->
