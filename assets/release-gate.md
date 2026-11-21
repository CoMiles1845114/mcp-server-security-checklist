# MCP release gate

Release/version: `<fill>`  
MCP specification baseline: `2025-06-18`  
Reviewer / rollback owner: `<fill locally; do not publish personal data>`

## Test envelope

- [ ] Written authorization exists for every tested endpoint.
- [ ] Disposable environment; no production credentials or personal data.
- [ ] Network starts denied or restricted to an explicit allowlist.
- [ ] Minimum filesystem/tool/OAuth privilege.
- [ ] Destructive handlers stubbed or protected by confirmation.
- [ ] Configuration snapshot, kill switch, and rollback path tested.

## Threat → control → evidence

| Threat | Control implemented | Evidence path / test ID | Pass / fail / N/A with reason |
|---|---|---|---|
| Confused deputy / redirect abuse |  |  |  |
| Token passthrough / wrong audience |  |  |  |
| Session theft or cross-user reuse |  |  |  |
| DNS rebinding / untrusted Origin |  |  |  |
| stdio protocol/log mixing |  |  |  |
| Excessive tool/resource scope |  |  |  |
| Prompt injection in metadata/results |  |  |  |
| Destructive action without confirmation |  |  |  |
| Secret/PII leakage in logs |  |  |  |
| Unpinned server/dependency artifact |  |  |  |
| Incident revocation and rollback |  |  |  |

## Uncovered items

List every applicable row without evidence. Zero scanner findings is not evidence that this section is empty.

## Decision

- [ ] **Release** — all applicable rows have evidence and no unresolved high-impact finding.
- [ ] **Hold** — owner and retest date recorded.
- [ ] **Rollback** — kill/revocation steps executed and verified.
