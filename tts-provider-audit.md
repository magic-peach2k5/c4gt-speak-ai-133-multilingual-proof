# Speak-AI TTS & G2P Provider Audit

## Current System

Speak-AI uses Kokoro TTS as its primary voice engine. Kokoro provides natural-sounding voices but has limited multilingual support out of the box.

### Current Architecture

```
User input (text)
    │
    ▼
Language detection / selection
    │
    ▼
G2P (Grapheme-to-Phoneme) conversion
    │
    ▼
Kokoro TTS inference
    │
    ▼
Audio output
```

### Limitations

| Area | Current State | Gap |
|---|---|---|
| Language coverage | English only (primary) | 10+ target languages not supported |
| G2P layer | Basic Latin-script support | No Arabic, Devanagari, Chinese character support |
| Model size | Kokoro ~200MB | Large for low-end devices |
| Latency | ~1-3s on desktop | Higher on low-end hardware |
| ONNX support | Not integrated | No optimised inference path |
| Caching | None | Every synthesis starts from scratch |

## Provider Evaluation

### Candidate TTS Providers

| Provider | Languages | Model Size | Latency | License | G2P Quality | Notes |
|---|---|---|---|---|---|---|
| **Kokoro (current)** | EN primary, limited others | ~200MB | Fast (GPU) | Apache 2.0 | Good (EN) | Baseline — used currently |
| **Piper TTS** | 20+ languages | ~10-50MB | Very fast | MIT | Good (multi) | Optimised for low-end; ONNX-native |
| **eSpeak-NG** | 100+ languages | ~5MB | Instant | GPL | OK (synthetic) | Lightest option; synthetic sound |
| **Coqui TTS** | 10+ languages | varies | Moderate | MIT | Good (multi) | Archived but models work |
| **Larynx** | 10+ languages | ~50-100MB | Fast | Apache 2.0 | Good | Trained on Common Voice |
| **Mimic III** | EN + limited | ~140MB | Moderate | Apache 2.0 | Good | Mycroft project |
| **Silero** | 10+ languages | ~50MB | Very fast | MIT | Good (multi) | ONNX-native; strong multilingual |

### Recommended Stack

```
Primary: Kokoro (existing — keep for English)
Fallback: Piper TTS (ONNX-native, 10-50MB, 20+ languages)
Secondary: eSpeak-NG (ultra-light, 100+ languages, synthetic quality)
```

## G2P Layer Analysis

### Current G2P

Kokoro uses a phoneme-based approach optimised for English. For non-Latin scripts, additional G2P processing is required.

### G2P Options by Language Family

| Language | Script | G2P Approach | Complexity | Existing Tools |
|---|---|---|---|---|
| Spanish | Latin | Rule-based + exception table | Low | Built into Piper |
| Portuguese (BR) | Latin | Rule-based | Low | Built into Piper |
| French | Latin | Rule-based + liaison rules | Medium | Built into Piper |
| Hindi | Devanagari | Unicode mapping + schwa deletion | Medium | Indic NLP library |
| Arabic | Arabic | Bidirectional + diacritics | High | pyarabic, farasa |
| Swahili | Latin | Rule-based | Low | Built into Piper |
| Quechua/Aymara | Latin | Rule-based | Low | Research G2P needed |
| Chinese (Mandarin) | Hanzi | Pinyin conversion required | High | pypinyin, jieba |
| Kinyarwanda | Latin | Rule-based | Low | Research needed |
| Guarani | Latin | Rule-based | Low | Research needed |

### Recommended G2P Strategy

```
- Latin-script languages (Spanish, Portuguese, French, Swahili): use Piper built-in G2P
- Devanagari (Hindi): extend G2P with Indic NLP library for schwa deletion
- Arabic: extend G2P with pyarabic + diacritic restoration
- Chinese: add pinyin conversion pipeline before TTS
- Low-resource (Quechua, Kinyarwanda, Guarani): rule-based fallback with eSpeak-NG
```

## Provider Abstraction Design

```python
class TTSProvider(ABC):
    @abstractmethod
    def synthesize(self, text: str, lang: str) -> bytes: ...
    @property
    def supported_languages(self) -> List[str]: ...
    @property
    def estimated_model_size_mb(self) -> float: ...
    @property
    def avg_latency_ms(self) -> int: ...

class KokoroProvider(TTSProvider): ...
class PiperProvider(TTSProvider): ...
class EspeakProvider(TTSProvider): ...

class TTSRouter:
    def __init__(self):
        self.providers = [KokoroProvider(), PiperProvider(), EspeakProvider()]
    
    def synthesize(self, text: str, lang: str) -> bytes:
        for provider in self.providers:
            if lang in provider.supported_languages:
                return provider.synthesize(text, lang)
        return self.fallback.synthesize(text, lang)
```

## References

- [Kokoro TTS GitHub](https://github.com/kokoro-tts) (current provider)
- [Piper TTS GitHub](https://github.com/rhasspy/piper) (recommended fallback)
- [eSpeak-NG](https://github.com/espeak-ng/espeak-ng) (ultra-light fallback)
- [Indic NLP Library](https://github.com/indic-nlp/indic-nlp-library) (Hindi G2P)
- [pyarabic](https://github.com/linuxscout/pyarabic) (Arabic G2P)
- [pypinyin](https://github.com/mozillazg/python-pypinyin) (Chinese G2P)
