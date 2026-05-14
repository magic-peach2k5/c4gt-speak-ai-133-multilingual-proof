# Speak-AI Latency & Low-End Device Performance Plan

## Performance Targets

| Metric | Target | Measurement |
|---|---|---|
| TTS latency (short phrase) | <500ms | Time from text input to audio ready |
| TTS latency (paragraph) | <2s | Time for 200 characters |
| Model load time | <5s | First synthesis after app start |
| Memory usage | <300MB | Peak during synthesis |
| CPU usage | <50% | Single-core during inference |
| App startup impact | <1s added | Time added to Sugar activity load |

## Optimisation Strategies

### 1. ONNX Runtime Integration

Convert Piper TTS models to ONNX format for:
- Faster CPU inference (2-4x improvement reported)
- Smaller memory footprint
- Cross-platform compatibility

```python
# Integration sketch
import onnxruntime as ort

class PiperONNXProvider:
    def __init__(self, model_path: str):
        self.session = ort.InferenceSession(model_path)
    
    def synthesize(self, text: str, lang: str) -> bytes:
        # Process through ONNX session
        audio = self.session.run(None, {"input": encoded_text})
        return audio
```

### 2. TTS Caching Strategy

Cache frequently synthesised phrases:

```python
class TTSLruCache:
    def __init__(self, max_size: int = 100, max_memory_mb: int = 50):
        self.cache = OrderedDict()
        self.max_size = max_size
        self.max_bytes = max_memory_mb * 1024 * 1024
        self.current_bytes = 0
    
    def get(self, key: str) -> Optional[bytes]:
        if key in self.cache:
            self.cache.move_to_end(key)
            return self.cache[key].audio
        return None
    
    def put(self, key: str, audio: bytes):
        while (len(self.cache) >= self.max_size or 
               self.current_bytes + len(audio) > self.max_bytes):
            # Evict least recently used
            old_key, old_val = self.cache.popitem(last=False)
            self.current_bytes -= len(old_val)
        self.cache[key] = audio
        self.current_bytes += len(audio)
```

**Cache keys:** `{language}:{text_hash[:16]}:{provider}`

### 3. Model Loading Strategy

| Provider | Load Strategy | Expected Time |
|---|---|---|
| Kokoro | Load on first English request | ~2s |
| Piper per-language | Load on first request per language | ~1s per voice |
| eSpeak-NG | Pre-load at startup | ~100ms |
| Piper voice switch | Pre-load next likely language in background | N/A |

### 4. Streaming Synthesis

For long texts, stream audio in chunks:

```python
def synthesize_streaming(self, text: str, lang: str, chunk_size: int = 50):
    """Synthesise text in chunks, yielding audio incrementally."""
    sentences = split_sentences(text)
    for sentence in sentences:
        audio = self.provider.synthesize(sentence, lang)
        yield audio
```

### 5. Low-End Device Profiles

| Profile | RAM | Storage | CPU | Recommended Settings |
|---|---|---|---|---|
| **Low** | 512MB | 4GB | Single-core 1GHz | eSpeak only, disable caching |
| **Medium** | 1GB | 8GB | Dual-core 1.5GHz | Piper light voices, 50-entry cache |
| **High** | 2GB+ | 16GB+ | Quad-core 2GHz+ | Kokoro + Piper, full caching |

## Battery Impact

| Provider | Estimated per-inference energy | Notes |
|---|---|---|
| eSpeak-NG | Very low (~0.1J) | Software synthesis |
| Piper ONNX | Low (~1J) | Optimised inference |
| Piper (non-ONNX) | Medium (~3J) | Full model inference |
| Kokoro | High (~5J) | Large model, GPU preferred |
