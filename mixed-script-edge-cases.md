# Speak-AI Mixed-Script & Edge Case Analysis

## 1. Mixed-Script Inputs

### Scenario: Mixed Latin + Non-Latin Text

| Input | Challenge | Handling Strategy |
|---|---|---|
| "Hello नमस्ते" | Two scripts, one string | Detect script boundaries, route each segment to appropriate provider |
| "العربية English" | RTL/LTR mixing | Bidirectional text handling required |
| "中文 and English" | CJK + Latin | Segment by Unicode block |
| "Français + Español" | Multiple Latin diacritics | All handled by same provider (Piper) |
| "¡Hola! ¿Cómo estás?" | Spanish punctuation | Piper handles inverted punctuation |

### Detection Algorithm

```python
import unicodedata

def detect_script_ranges(text: str) -> List[Tuple[int, int, str]]:
    """Return (start, end, script_name) for each script segment."""
    ranges = []
    current_start = 0
    current_script = None
    
    for i, char in enumerate(text):
        script = unicodedata.name(char, "").split()[0] if char else "UNKNOWN"
        # Simplify to major script categories
        if char in "¿¡":
            script = "LATIN"  # Spanish punctuation
        script_category = categorize_script(script)
        
        if script_category != current_script:
            if current_script is not None:
                ranges.append((current_start, i, current_script))
            current_start = i
            current_script = script_category
    
    if current_script is not None:
        ranges.append((current_start, len(text), current_script))
    return ranges
```

## 2. Special Character Handling

| Character Type | Example | Handling |
|---|---|---|
| Emoji | "Hello 😊 world" | Filter out, preserving text order |
| URLs | "Visit https://example.com" | Speak as "visit example dot com" |
| Email | "Contact us@test.org" | Speak as "contact us at test dot org" |
| Hashtags | "#MondayMotivation" | Speak as "hashtag Monday Motivation" |
| Numbers | "100°C, 50%" | Localise number format per language |
| Currency | "$1,234.56 or €500" | Speak currency symbol + amount |
| Dates | "2026-05-14" | Localise: "May 14, 2026" or "14 de mayo" |
| Time | "14:30 or 2:30 PM" | Localise per language convention |
| Math | "x² + y² = z²" | Speak as "x squared plus y squared" |
| HTML entities | "&amp; &lt; &gt;" | Decode before synthesis |

## 3. Unsupported Language / Missing Voice

### Detection

```python
class UnsupportedLanguageError(Exception):
    pass

def check_language_supported(lang: str, available: List[str]) -> bool:
    """Check if language has ANY provider."""
    return any(lang in p.supported_languages 
               for p in PROVIDER_CHAIN)
```

### User Experience

When a language is not supported:
1. Show language selector with clear supported/unsupported indicators
2. If user selects unsupported language, show message:
   `"Language not yet available. Currently supported: [list]"`
3. Option to hear eSpeak-NG synthesised version with quality warning

## 4. Offline / No-Provider State

| Scenario | Handling |
|---|---|
| No TTS providers installed | App still opens; audio buttons disabled with tooltip |
| Model file deleted after first use | Catch load error, fall back to next provider |
| Partial provider install (language subset) | Only show installed languages in selector |
| All providers fail at runtime | Fallback: display text in a speech-bubble UI without audio |

## 5. Input Validation

| Input Type | Example | Handling |
|---|---|---|
| Empty string | "" | Return silence (160ms of 0s) |
| Whitespace only | "   " | Same as empty |
| Very long (>5000 chars) | 10000 chars | Truncate to 5000 with warning |
| Non-printable characters | "\x00\x01\x02" | Filter non-printable, log warning |
| HTML/markup | "<b>bold</b>" | Strip tags before synthesis |
| Only punctuation | "!!!???!!!" | Return silence, log debug |

## 6. Concurrent Access & Thread Safety

| Scenario | Handling |
|---|---|
| Two syntheses simultaneous | Pipeline: queue, process sequentially per provider |
| Rapid language switching | Cancel in-flight synthesis, start new |
| App backgrounded during synthesis | Allow current synthesis to finish, then release model |
| Activity destroyed during loading | Cancel via coroutine/thread lifecycle |

## 7. Output Validation

| Check | Method | Action |
|---|---|---|
| Audio duration sanity | Expected_duration ± 50% | Log warning if outside range |
| Audio too short (<10ms for 200 chars) | Duration check | Re-run with fallback provider |
| Audio too long (>30s for 200 chars) | Duration check | Log error, truncate |
| Empty audio bytes | Byte length check | Retry once with next provider |
| Distorted audio | Sample range check | Log quality warning, do not block |
