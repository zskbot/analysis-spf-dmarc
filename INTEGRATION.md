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
