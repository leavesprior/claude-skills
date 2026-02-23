# PentAGI Security Audit

**Date**: 2026-02-22
**Repository**: `github.com/vxcontrol/pentagi`
**Audit Type**: READ-ONLY static analysis, configuration review, architecture assessment

## Summary

PentAGI is an autonomous AI-powered pentesting platform with sophisticated multi-agent architecture and 4-tier memory. We audited it before considering any integration with Neoma's guardian agent.

**Verdict**: DO NOT integrate wholesale. Extract architecture patterns only.

## Key Findings

- **4 CRITICAL**: Docker root+socket (CVSS 10.0), NET_ADMIN, default credentials, no SSL
- **6 HIGH**: Unvetted vendor deps, untrusted Docker image, loose pinning, plaintext creds
- **7 MEDIUM**: No RLS, no rate limiting, SSRF risk, unencrypted screenshots
- **4+ LOW**: Host binding defaults, CORS, logging

## What We Extracted for Guardian

1. **Episodic memory pattern** -> `neoma-guardian-memory` (store/query/report/patterns)
2. **Self-scan pattern** -> `neoma-guardian-self-scan` (port/service baseline verification)
3. **Multi-agent coordination** -> Flagpole routing for `security_audit` task type

## Files

- `AUDIT_REPORT.md` - Full detailed audit (1740 lines)
- `EXECUTIVE_SUMMARY.txt` - Executive summary with priority matrix

## Neoma Integration

- Guardian episodic memory: MB namespace `guardian_episodic`
- Self-scan timer: daily 04:00 via systemd
- Wheelwright tapes: `guardian.self-scan.json`, `guardian.episodic-memory.json`
- Source audit (read-only): `~/Neoma_project/security_research/pentagi-audit/`
