# Speak-AI Language Support Matrix

## Target Languages

| # | Language | Script | Speaker Count | Priority | Current Status |
|---|---|---|---|---|---|
| 1 | Spanish | Latin | 500M+ | P0 | Supported by Piper, eSpeak-NG |
| 2 | Portuguese (BR) | Latin | 250M+ | P0 | Supported by Piper, eSpeak-NG |
| 3 | Hindi | Devanagari | 600M+ | P0 | G2P needed (Indic NLP) |
| 4 | French | Latin | 300M+ | P0 | Supported by Piper, eSpeak-NG |
| 5 | Arabic | Arabic | 370M+ | P1 | G2P needed (pyarabic) |
| 6 | Swahili | Latin | 100M+ | P1 | Supported by Piper, eSpeak-NG |
| 7 | Quechua/Aymara | Latin | 10M+ | P2 | Limited TTS support |
| 8 | Chinese (Mandarin) | Hanzi | 1.2B+ | P1 | G2P needed (pypinyin) |
| 9 | Kinyarwanda | Latin | 15M+ | P2 | Limited TTS support |
| 10 | Guarani | Latin | 7M+ | P2 | Limited TTS support |

## Provider Coverage

| Language | Kokoro | Piper TTS | eSpeak-NG | Notes |
|---|---|---|---|---|
| Spanish | Partial | ✅ Good | ✅ Basic | Piper recommended as primary |
| Portuguese (BR) | ❌ | ✅ Good | ✅ Basic | Piper recommended |
| Hindi | ❌ | ✅ (via XL) | ✅ Basic | Requires G2P pipeline |
| French | Partial | ✅ Good | ✅ Basic | Piper recommended |
| Arabic | ❌ | ❌ | ✅ Basic | eSpeak + research model |
| Swahili | ❌ | ✅ | ✅ Basic | Piper recommended |
| Quechua/Aymara | ❌ | ❌ | ✅ Basic | eSpeak only |
| Chinese | ❌ | ❌ | ✅ Basic | Requires pinyin + research |
| Kinyarwanda | ❌ | ❌ | ✅ Basic | eSpeak only |
| Guarani | ❌ | ❌ | ✅ Basic | eSpeak only |

## Quality Targets

| Tier | Quality Level | Languages | Provider |
|---|---|---|---|
| **High quality** | Natural, human-like | ES, PT-BR, FR, SW | Piper TTS |
| **Medium quality** | Acceptable with noticeable TTS | HI, AR | Piper + G2P |
| **Low quality** | Synthetic but understandable | ZH | eSpeak + pinyin |
| **Minimal** | Robotic but functional | QU, RW, GN | eSpeak-NG |

## Implementation Plan

### Phase 1 (Mid-point milestone)
- [x] Audit current TTS/G2P layer
- [x] Evaluate replacement/fallback providers
- [ ] Integrate Piper TTS for Spanish, Portuguese, French
- [ ] Add Hindi G2P pipeline (Indic NLP)
- [ ] Basic language selector UI update

### Phase 2
- [ ] Add Arabic G2P pipeline
- [ ] Add Swahili via Piper
- [ ] Implement TTSRouter with fallback chain
- [ ] Pronunciation test pass for Phase 1 languages

### Phase 3 (Stretch)
- [ ] Chinese pinyin + TTS pipeline
- [ ] Low-resource language support (Quechua, Kinyarwanda, Guarani)
- [ ] ONNX runtime integration for faster inference
- [ ] TTS model caching for common phrases
