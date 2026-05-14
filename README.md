# Speak-AI #133 Proof Packet

Purpose: prove multilingual TTS understanding before final DMP submission. This replacement project replaces P3 Medic #10707.

## Artifacts

- [`tts-provider-audit.md`](tts-provider-audit.md) — Current TTS/G2P audit + provider evaluation (7 providers)
- [`language-support-matrix.md`](language-support-matrix.md) — 10-language matrix with provider coverage and quality tiers
- [`fallback-provider-design.md`](fallback-provider-design.md) — Provider chain architecture, error handling, config
- [`pronunciation-test-matrix.md`](pronunciation-test-matrix.md) — 14 pronunciation + 10 latency + 10 edge-case tests
- [`latency-low-end-device-plan.md`](latency-low-end-device-plan.md) — ONNX, caching, streaming, device profiles
- [`mixed-script-edge-cases.md`](mixed-script-edge-cases.md) — Mixed-script detection, special chars, input validation

## Reviewer Boundary

This is a proof package and provider design, not accepted upstream work yet. No upstream PR is open at submission time. Claim boundary: local TTS/provider design proof only.
