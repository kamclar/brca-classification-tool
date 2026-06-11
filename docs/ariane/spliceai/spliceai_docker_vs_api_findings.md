# SpliceAI Reference Transcript Pilot Validation Summary

Date: 2026-06-11

## Scope

This validation checks whether locally computed Broad-compatible SpliceAI
scores from the patched Docker endpoint match the public Broad SpliceAI Lookup
API when both are interpreted on the BRCA ENIGMA reference transcripts.

Reference transcripts:

- BRCA1: ENST00000357654.9 / NM_007294.4
- BRCA2: ENST00000380152.8 / NM_000059.4

Policy:

- `reference_transcript`
- Score is max of DS_AG, DS_AL, DS_DG, DS_DL within the reference transcript
  record only.

## Local Pilot

Local endpoint:

- `http://localhost:8080/spliceai/`
- Docker image: `docker.io/weisburd/spliceai-38:latest`
- Patched wrapper supports `transcript=...`

Local cache after pilot:

- 100 variants with SpliceAI scores
- 1172 variants still missing from SpliceAI cache but already having GRCh38
  coordinates
- 46275 variants still missing coordinates

Summary file:

- `variant_space_scan/outputs/brca_snv_manifest.with_spliceai.ref_pilot100.summary.json`

## Public API Validation

Validation command:

```powershell
python variant_space_scan\validate_spliceai_reference_pilot.py `
  --sample-size 100 `
  --sleep-seconds 8 `
  --timeout 180 `
  --retries 1 `
  --retry-sleep-seconds 30 `
  --report variant_space_scan\outputs\spliceai_reference_pilot_validation.100_slow.json
```

Initial public API validation:

- 96 variants validated directly
- 4 variants failed due to public API timeout or HTTP 500
- 0 numeric mismatches among validated public responses

The 4 failed public API calls were retried manually with longer pauses and
timeout. All 4 matched the local reference transcript scores:

- BRCA1:c.34C>A: local 0.02, public 0.02
- BRCA1:c.9A>T: local 0.01, public 0.01
- BRCA1:c.15T>G: local 0.03, public 0.03
- BRCA1:c.8T>C: local 0.04, public 0.04

Final interpretation:

- 100/100 local reference transcript SpliceAI scores matched public Broad API
  reference transcript scores.
- Public API failures were availability or timeout issues, not score
  discrepancies.

## Conclusion

The patched local Docker endpoint with `transcript=...` is API-equivalent for
the tested BRCA1 reference transcript variants when the public Broad API output
is interpreted on the same reference transcript.

This supports using local Docker SpliceAI for precomputing the BRCA
reference-transcript cache, with occasional public Broad API spot checks.
