# MCP Server Pre-Release Security Checklist


> **Decision:** Use this checklist before releasing or connecting an MCP server with access to files, credentials, networks, or destructive tools; do not treat it as a security certification or a substitute for implementation-specific testing.
>
> **For:** MCP server maintainers, client integrators, and security reviewers  ·  **Not for:** teams seeking a compliance badge or live-target attack playbook

Use this gate before exposing an MCP server to users or connecting a client that can reach files, credentials, network services, or destructive tools. It is a **release checklist**, not a security certification.

**Frozen baseline:** MCP specification and security guidance dated **2025-06-18**; OAuth reference is the Internet-Draft at the URL reviewed on **2026-11-21**. Re-run this review whenever the implementation targets a newer protocol/auth draft.

## Safe test envelope — establish this first

- Test only a server/client you own or are explicitly authorized to assess.
- Use a disposable host/container and fake credentials with zero production privilege.
- Start with loopback or an isolated network; allowlist only endpoints the test requires.
- Give the client and server minimum filesystem, tool, OAuth, and network scopes.
- Keep destructive tool handlers stubbed until confirmation and policy tests pass.
- Snapshot configuration and define a rollback/kill procedure before connecting an Inspector or scanner.
- Never point broad scanners at third-party MCP endpoints without permission.

## Release gate

Copy [`assets/release-gate.md`](assets/release-gate.md) into the release evidence. A row passes only when it names a control **and** an artifact proving that control.

| Threat / release question | Required control | Evidence to retain | What remains unproven |
|---|---|---|---|
| Confused deputy during OAuth | Bind authorization to the intended client/resource; validate redirect URIs and state/PKCE as required by the chosen flow | Redacted authorization trace, negative redirect/state tests, configured resource/audience | IdP correctness or future draft changes |
| Token passthrough | Server accepts only tokens issued for itself; never forward a client token as an upstream token | Audience/resource negative test and code/config reference | Upstream service authorization quality |
| Session hijack/fixation | Unpredictable session identifiers, user/session binding, expiry, and no use of session ID as authentication | Two-user isolation test; expired/foreign session rejection | Host compromise |
| Local HTTP / DNS rebinding | Validate `Origin` where applicable; bind local servers to loopback unless intentionally remote; authenticate every connection | Bind address, Origin rejection test, firewall rule | Browser/OS vulnerabilities |
| stdio transport boundary | Protocol messages stay on stdout; logs/errors go to stderr; child process inherits no unnecessary secrets | Captured clean protocol stream and sanitized environment list | Safety of invoked tools |
| Tool/resource overbreadth | Explicit allowlist; minimum path/network scope; per-user authorization; default deny unknown names/arguments | Enumerated exposed surface and denied out-of-scope calls | Correctness of every allowed tool |
| Untrusted tool descriptions/results | Treat names, descriptions, resources, prompts, and tool output as untrusted data; do not turn them into authority | Injection fixtures and action-interception logs | Unknown model failure modes |
| Destructive or costly action | Human confirmation at the action boundary; show concrete target/arguments; idempotency or dry-run where possible | Confirmation test, cancellation test, retry/idempotency test | Human judgment quality |
| Secret or personal data in logs | Structured allowlist logging; redact tokens, headers, prompts/results where policy requires; access and retention limits | Sample redacted log, retention setting, access list | Leakage outside the logging path |
| Dependency/server substitution | Pin reviewed artifact/version and origin; verify package/image/signature where available | Lockfile or digest and provenance record | That reviewed code is vulnerability-free |
| Incident containment | Kill switch, token revocation, session invalidation, audit trail, and rollback owner | Timed rollback drill and contact/runbook | Recovery from every downstream effect |

**Stop release** if any row that matches the server’s exposure lacks evidence. “Inspector connected” or “scanner returned zero” is never sufficient release evidence.

## Minimal abuse cases

Run these in the safe envelope with fake data:

1. A second user tries another user’s session ID.
2. A token with the wrong audience/resource reaches the server.
3. A browser request carries an unapproved `Origin` to a loopback HTTP transport.
4. A tool description/resource contains an instruction to reveal a fake secret or invoke another tool.
5. A tool call requests a path/host/argument outside the allowlist.
6. A destructive call is retried, cancelled, and submitted without confirmation.
7. A fake token appears in an error path; verify it is absent from retained logs.
8. The server is stopped mid-operation; verify rollback or a clearly recoverable state.

## Inspector and scanner procedure

1. Launch the disposable server with fake credentials and minimum permissions.
2. Record the expected tools/resources/prompts before connecting an Inspector.
3. Connect the official [MCP Inspector](https://github.com/modelcontextprotocol/inspector) only to that authorized endpoint. A community inspector or [mcp-scan](https://github.com/invariantlabs-ai/mcp-scan) may add observations, but neither is an authority to release.
4. Diff the observed surface against the allowlist; investigate every unexpected capability.
5. Destroy the environment, revoke test tokens, and retain only redacted evidence.

## Versioned authority and supporting guidance

- Normative protocol areas: [MCP 2025-06-18 repository/specification](https://github.com/modelcontextprotocol/modelcontextprotocol), including authorization and transports.
- MCP-specific attacks and controls: [versioned security best practices](https://raw.githubusercontent.com/modelcontextprotocol/modelcontextprotocol/main/docs/docs/2025-06-18/tutorials/security/security_best_practices.mdx).
- Authorization context: [OAuth 2.1 draft](https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/); freeze the draft version used by an implementation rather than assuming a moving draft is stable.
- General threat prompts: [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/).
- Governance overlay: [NIST AI RMF Generative AI Profile](https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence).
- Optional development UI: [MCPJam Inspector](https://github.com/MCPJam/inspector).

OWASP and NIST broaden threat/governance coverage; they do **not** replace the MCP-specific rows above. Tool homepages establish tool roles only. The control mapping and stop-release rule are **author recommendations in this checklist**, not claims made by all linked sources.

## Maintenance

Re-review when the targeted MCP spec/auth draft changes, a transport is added, a tool gains privilege, a new identity provider is introduced, or the exposure changes from local to remote. Keep evidence versioned with the release; never update a date without rerunning the relevant row.

Source ledger: [`sources/sources.json`](sources/sources.json). Release artifact: [`assets/release-gate.md`](assets/release-gate.md).

## Adjacent guides

- [repo-agent-prompt-injection-tests](https://github.com/Ozekmiami/repo-agent-prompt-injection-tests) — Use it when repository content can steer a coding agent and you need inert prompt-injection fixtures.
- [agent-execution-control-sandbox](https://github.com/MargaritaOfficial/agent-execution-control-sandbox) — Use it when tool approval is not enough and the agent also needs an enforced execution boundary.
- [dependency-agent-safety-gates](https://github.com/WILLIAMSTONSA/dependency-agent-safety-gates) — Use it when an agent proposes dependency changes that need independent evidence and rollback.

## License

Original checklist and release-gate template are available under CC BY 4.0. Linked standards and projects retain their own terms.

If this saved you a decision, star the repo — it is how the next reader finds it.
