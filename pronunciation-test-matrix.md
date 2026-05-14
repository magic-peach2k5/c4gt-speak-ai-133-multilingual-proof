# Speak-AI Pronunciation & Latency Test Matrix

## 1. Pronunciation Quality Tests

### Test Methodology

For each language, test:
- **Single word pronunciation** — 10 common words read by native speaker reference vs TTS output
- **Sentence prosody** — 5 sentences with varied intonation (question, statement, exclamation)
- **Numbers and dates** — 5 number/date formats
- **Proper nouns** — 5 local names/places
- **Minimal pairs** — 5 pairs distinguished by single phoneme

Scoring: 1 (unintelligible) to 5 (native-like)

### Test Cases

| # | Language | Test Phrase | Script | Expected | Current Score | Notes |
|---|---|---|---|---|---|---|
| PR-01 | Spanish | "Buenos días, ¿cómo estás?" | Latin | Clear greeting intonation | TBD | Compare Kokoro vs Piper |
| PR-02 | Spanish | "Gracias por su ayuda" | Latin | Soft 'c', correct stress | TBD |  |
| PR-03 | PT-BR | "Bom dia, tudo bem?" | Latin | Nasal vowels | TBD | Piper should handle well |
| PR-04 | PT-BR | "Obrigado pela atenção" | Latin | 'lh' and 'nh' sounds | TBD |  |
| PR-05 | Hindi | "नमस्ते, आप कैसे हैं?" | Devanagari | Schwa deletion | TBD | G2P quality critical |
| PR-06 | Hindi | "धन्यवाद" | Devanagari | Correct consonant clusters | TBD |  |
| PR-07 | French | "Bonjour, comment allez-vous?" | Latin | Liaison, nasal vowels | TBD | Piper expected best |
| PR-08 | French | "Je ne sais pas" | Latin | 'ne' elision | TBD |  |
| PR-09 | Arabic | "السلام عليكم" | Arabic | Pharyngeal consonants | TBD | eSpeak baseline |
| PR-10 | Arabic | "شكراً جزيلاً" | Arabic | Correct 'ain, ghain | TBD | Challenging for TTS |
| PR-11 | Swahili | "Habari za asubuhi" | Latin | Clear vowel sounds | TBD | Piper should handle |
| PR-12 | Swahili | "Asante sana" | Latin | Stress on penultimate | TBD |  |
| PR-13 | Chinese | "你好，今天天气很好" | Hanzi | Tone accuracy | TBD | Pinyin G2P critical |
| PR-14 | Chinese | "谢谢你的帮助" | Hanzi | Third tone sandhi | TBD |  |

### Scoring Rubric

| Score | Label | Description |
|---|---|---|
| 5 | Native-like | Indistinguishable from native speaker |
| 4 | Good | Minor accent, fully intelligible |
| 3 | Acceptable | Noticeable accent, mostly intelligible |
| 2 | Poor | Difficult to understand, frequent errors |
| 1 | Unintelligible | Cannot understand the output |

## 2. Latency Tests

### Test Cases

| # | Scenario | Provider | Input Length | Target Latency | Measured |
|---|---|---|---|---|---|
| LT-01 | Short phrase | Kokoro | 10 chars | <1s | TBD |
| LT-02 | Short phrase | Piper | 10 chars | <500ms | TBD |
| LT-03 | Short phrase | eSpeak | 10 chars | <100ms | TBD |
| LT-04 | Paragraph | Kokoro | 200 chars | <3s | TBD |
| LT-05 | Paragraph | Piper | 200 chars | <2s | TBD |
| LT-06 | Paragraph | eSpeak | 200 chars | <500ms | TBD |
| LT-07 | Long text | Kokoro | 1000 chars | <10s | TBD |
| LT-08 | Long text | Piper | 1000 chars | <5s | TBD |
| LT-09 | First load | Piper (cold) | N/A | <2s model load | TBD |
| LT-10 | Cache hit | Kokoro (cached) | 50 chars | <200ms | TBD |

### Latency Budget

```
Total latency target: <2-3s (input to audio)

Breakdown:
├── G2P conversion: <100ms
├── Provider selection: <50ms
├── TTS inference:
│   ├── Piper: <800ms (short), <2s (paragraph)
│   ├── eSpeak: <100ms (any)
│   └── Kokoro: <1.5s (short), <3s (paragraph)
└── Audio playback: <100ms
```

## 3. Mixed-Script & Edge Cases

| # | Scenario | Input | Expected Behaviour |
|---|---|---|---|
| EC-01 | Mixed Latin/Devanagari | "Hello नमस्ते" | Detect primary script, fall back per word |
| EC-02 | Numbers in Arabic text | "رقم ١٢٣" | Handle Arabic-Indic digits |
| EC-03 | URL in text | "Visit example.com" | Skip or speak letter by letter |
| EC-04 | Punctuation-heavy | "¡Hola! ¿Cómo estás?" | Correct intonation for ¡ ¿ |
| EC-05 | Empty string | "" | Return silence, no crash |
| EC-06 | Very long input | 5000 chars | Truncate or stream |
| EC-07 | Emoji | "Hello 😊" | Skip emoji, speak text |
| EC-08 | Abbreviations | "Dr. Smith (MD)" | Expand or speak as-is |
| EC-09 | Language switch mid-sentence | "Hello in Spanish is hola" | Detect language boundaries |
| EC-10 | Special characters | "100°C, 50% off, #1" | Handle symbols appropriately |

## 4. Low-End Device Tests

| # | Scenario | Device Profile | Expected Behaviour |
|---|---|---|---|
| LE-01 | Model loading | 1GB RAM, slow storage | Load under 5s, show loading indicator |
| LE-02 | Inference memory | 1GB RAM | <300MB memory during synthesis |
| LE-03 | Concurrent requests | 2 syntheses overlapping | Queue second until first completes |
| LE-04 | Background app | Low memory state | Release TTS model, reload on demand |
| LE-05 | CPU-only inference | No GPU/accelerator | Piper ONNX should run on CPU |
