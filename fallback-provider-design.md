# Speak-AI Fallback Provider Architecture

## Design Goal

Route each language to the best available TTS provider, with automatic fallback when higher-quality providers are unavailable.

## Provider Chain

```
User selects language L with text T
        │
        ▼
TTSRouter.route(L, T)
        │
        ├── Kokoro supports L? → Kokoro.synthesize(T, L)
        │       │
        │       └── Error? → Fallback
        │
        ├── Piper supports L? → Piper.synthesize(T, L)
        │       │
        │       └── Error? → Fallback
        │
        └── eSpeak-NG supports L? → eSpeak.synthesize(T, L)
                │
                └── Error? → User-visible error
```

## Error Handling Strategy

### Detection

| Layer | Detection Method | Action |
|---|---|---|
| Provider not installed | ImportError at init | Skip provider chain |
| Model file missing | FileNotFoundError at load | Log warning, skip provider |
| Language not supported | ValueError at synthesize | Try next provider |
| Inference failure | RuntimeError during synthesis | Log error, try next provider |
| Output timeout | Duration > 10s threshold | Log warning, return partial |

### Fallback Logic

```python
def synthesize_with_fallback(self, text: str, lang: str) -> bytes:
    errors = []
    for provider in [self.kokoro, self.piper, self.espeak]:
        try:
            if provider.supports(lang):
                return provider.synthesize(text, lang)
        except Exception as e:
            errors.append((provider.name, str(e)))
            continue
    # All providers failed
    raise NoProviderAvailableError(
        f"No working provider for {lang}. Tried: {errors}"
    )
```

## Configuration File

```json
{
  "providers": {
    "kokoro": {
      "enabled": true,
      "model_path": "models/kokoro/kokoro_v1.pt",
      "priority": 1
    },
    "piper": {
      "enabled": true,
      "model_dir": "models/piper/",
      "priority": 2,
      "voices": {
        "es": "models/piper/es_ES.xmalingua",
        "pt_BR": "models/piper/pt_BR.xmalingua",
        "fr": "models/piper/fr_FR.xmalingua",
        "sw": "models/piper/sw_TZ.xmalingua"
      }
    },
    "espeak": {
      "enabled": true,
      "priority": 3,
      "language_map": {
        "es": "es",
        "pt": "pt",
        "fr": "fr",
        "hi": "hi",
        "ar": "ar",
        "sw": "sw",
        "qu": "qu",
        "zh": "cmn",
        "rw": "rw",
        "gn": "gn"
      }
    }
  }
}
```

## Provider Loading Strategy

```
App start
    │
    ▼
TTSRouter.init()
    │
    ├── Try import kokoro → set kokoro = KokoroProvider() or None
    ├── Try import piper → set piper = PiperProvider() or None
    └── Try import espeak → set espeak = EspeakProvider() or None
    │
    ▼
ready = [p for p in [kokoro, piper, espeak] if p is not None]
if not ready:
    log critical: No TTS providers available
```

## Memory Management

| Provider | Loaded | Memory | Strategy |
|---|---|---|---|
| Kokoro | On demand | ~200MB | Load when first EN request; keep warm |
| Piper | On demand | ~10-50MB per voice | Load per language; cache one voice |
| eSpeak-NG | Always | ~5MB | Pre-load at startup |

## Performance Targets

| Metric | Target | Measurement |
|---|---|---|
| Fallback transition latency | <500ms | Time from provider failure to next provider |
| First-request latency (Piper) | <2s | Cold start per language |
| Subsequent latency (Piper) | <500ms | Warm inference |
| eSpeak latency | <100ms | Always fast |
| Memory budget | <250MB total | Kokoro + Piper + eSpeak |
