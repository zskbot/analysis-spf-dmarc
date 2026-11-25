# Security Center integration contract — analysis-spf-dmarc

## Audit snapshot
- Python CLI/domain analyzer with SPF and DMARC parsing, DNS queries via dnspython, Tranco integration, requests, `dmarc` and `pyspf`.
- Dependencies are listed in `config/requirements.txt`; several are version-ranged rather than fully locked.
- README documents SPF recursion/lookup counting, DMARC policy/RUA/RUF parsing, attack indicators and a 0–100 risk score.
- README disclaimer limits use to supervised research, education and authorized domain audits. It explicitly excludes malicious attacks and unauthorized scans.

## Integration role
Email-authentication posture analyzer. Run only against domains the requester owns or is explicitly authorized to assess.

## Preview API contract
- `POST /v1/intel/email-auth/check` with a normalized domain and explicit authorization context.
- Return SPF record and lookup count, DMARC policy/tags, DNS observations, findings, evidence, risk score and check timestamp.
- Keep raw DNS evidence and derived findings distinct. A missing record is not automatically proof of spoofing.

## Guardrails and blockers
- Enforce explicit authorization, per-domain rate limits and bounded DNS recursion/timeouts.
- Do not send mail, alter DNS or publish SPF/DMARC changes.
- Add dependency lock/pinning and tests for DNS timeout, NXDOMAIN, malformed records, SPF lookup-limit and DMARC policy edge cases.
- No bulk scans of third-party domains.
- This adapter is read-only and Preview-only.


## Critical source exposure discovered during audit
- The default branch tracks `default.private`, which contains a PEM private-key payload. The key material is intentionally not reproduced here.
- `dkim_sign.py` reads a private key and constructs an email with multiple `From` headers, including a spoofed display identity, then invokes `swaks` to send it. This script is excluded from the Security Center integration and must never be exposed as a public API or run by Preview automation.
- Before merging any remediation, the key owner must identify the selector/domain, revoke/rotate the DKIM key and update DNS through an explicitly approved process. This task makes no DNS changes. Deleting the file from a branch does not erase it from Git history; a history purge and verification are separate follow-up operations.
- Proposed branch cleanup removes the tracked private-key file and the mail-sending proof-of-concept script, and adds private-key patterns to `.gitignore`. The key must still be treated as compromised until rotated.


## Required credential-incident remediation (not completed by this PR)
1. **Revoke/rotate first:** identify the DKIM selector and domain owner; generate a new private/public DKIM key pair in a trusted environment; publish the new public key in DNS; verify DKIM signing and DMARC alignment; then revoke the exposed key. Until this is complete, assume the exposed key is compromised. Do not paste private key material into GitHub issues, PRs or chat.
2. **Purge Git history separately:** after the replacement key is active, repository administrators must approve a coordinated history rewrite. Use `git filter-repo` to remove `default.private` from every reachable ref (and assess whether the sender script should also be purged); force-push rewritten branches/tags, coordinate all collaborators to re-clone or hard-reset, invalidate cached artifacts/forks where possible, and verify all reachable refs no longer contain the key. A normal commit deletion or this PR's file removal is not a history purge.
3. **Verify exposure:** review access/audit logs and any CI artifacts, releases, caches, forks or copies that may contain the key. Rotate any other credentials if evidence shows reuse or exposure.
4. **Approval boundary:** this PR intentionally does not rewrite `main` history, force-push refs, change DNS, or rotate keys. Those actions require domain/repository-owner authorization and a coordinated maintenance window.
