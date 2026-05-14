# Speak-AI #133 Proof Packet

Purpose: prove multilingual TTS understanding before final DMP submission. This replacement project replaces P3 Medic #10707.

## Artifacts

- [`tts-provider-audit.md`](tts-provider-audit.md) — Current TTS/G2P audit + provider evaluation (7 providers)
- [`language-support-matrix.md`](language-support-matrix.md) — 10-language matrix with provider coverage and quality tiers
- [`fallback-provider-design.md`](fallback-provider-design.md) — Provider chain architecture, error handling, config
- [`pronunciation-test-matrix.md`](pronunciation-test-matrix.md) — 14 pronunciation + 10 latency + 10 edge-case tests
- [`latency-low-end-device-plan.md`](latency-low-end-device-plan.md) — ONNX, caching, streaming, device profiles
- [`mixed-script-edge-cases.md`](mixed-script-edge-cases.md) — Mixed-script detection, special chars, input validation

## Screenshots

- [`screenshots/tts-architecture-flow.png`](screenshots/tts-architecture-flow.png) — End-to-end pipeline: text comes in, the language router figures out what it is, the G2P layer normalises the script, the TTSRouter picks the best provider (Kokoro, Piper, or eSpeak), and audio comes out.
- [`screenshots/language-selector-ui.png`](screenshots/language-selector-ui.png) — The language picker. 10 languages grouped by priority tier (P0-P2), auto-detection running, and a live provider status bar showing which engine is active.
- [`screenshots/proof-data-matrix.png`](screenshots/proof-data-matrix.png) — Three proof tables in one view: the full language-coverage matrix across Kokoro/Piper/eSpeak, the 7-provider audit with the selection rationale, and the implementation phase breakdown.
- [`screenshots/provider-status-panel.png`](screenshots/provider-status-panel.png) — Live provider dashboard. Shows each engine's status, supported languages, model size, latency, and performance bars for latency budget and memory usage.
- [`screenshots/pronunciation-test-screen.png`](screenshots/pronunciation-test-screen.png) — The full test plan rendered as tables: 14 pronunciation tests with scores per language, 10 latency test results with budget checks, and 10 edge-case scenarios.
- [`screenshots/fallback-warning.png`](screenshots/fallback-warning.png) — What the user sees when Piper is unavailable for a language. Shows the fallback chain stepping through Piper (fail) -> eSpeak (active), details the error, and offers retry or continue.
- [`screenshots/low-end-device-mode.png`](screenshots/low-end-device-mode.png) — The low-end device optimisation panel. Hardware profile detected (RPi 3 / 1GB RAM), toggle switches for each optimisation (eSpeak-only, cache, streaming), and estimated performance numbers.

## Reviewer Boundary

This is a proof package and provider design, not accepted upstream work yet. No upstream PR is open at submission time. Claim boundary: local TTS/provider design proof only.
