# Ollama Integration Analysis for MARK XXV

## Executive Summary

**Status:** The codebase CAN be modified to use Ollama instead of Gemini, but there are **significant architectural challenges** that require substantial modifications, particularly around the live audio streaming functionality.

**Recommendation:** Consider a hybrid approach or use Kimi k2.5 via Moonshot AI API rather than Ollama for better feature parity.

---

## Current Gemini Usage Breakdown

| Component | Purpose | Model Used | Complexity to Replace |
|-----------|---------|------------|----------------------|
| `main.py` | Live audio streaming, function calling | `gemini-2.5-flash-native-audio-preview` | **HIGH** |
| `agent/planner.py` | Task planning | `gemini-2.5-flash-lite` | Low |
| `agent/executor.py` | Code generation, translation, summarization | `gemini-2.5-flash` | Low |
| `agent/error_handler.py` | Error analysis, fix generation | `gemini-2.5-flash-lite` | Low |
| `actions/screen_processor.py` | Vision + audio analysis | `gemini-2.5-flash-native-audio-preview` | **HIGH** |
| `actions/web_search.py` | Web search with Google grounding | `gemini-2.5-flash-lite` | Medium |
| `actions/*.py` (various) | Text generation | Various | Low |

---

## Key Challenges with Ollama Integration

### 1. Live Audio Streaming (CRITICAL)

**Current Implementation:**
```python
# main.py:43
LIVE_MODEL = "models/gemini-2.5-flash-native-audio-preview-12-2025"
```

The main JARVIS experience relies on Gemini's **Live API** for real-time bidirectional audio streaming:
- Audio input from microphone streamed directly to Gemini
- Audio output (TTS) received directly from Gemini in the response stream
- Latency: ~300-500ms end-to-end

**Ollama Limitation:**
- Ollama does **NOT** support native audio streaming
- No built-in speech-to-text (STT) or text-to-speech (TTS)
- Requires implementing a separate pipeline: **Audio → STT → LLM → TTS → Audio**

**Required Changes:**
- Add STT service (e.g., Whisper, Vosk, Windows Speech Recognition)
- Add TTS service (e.g., pyttsx3, Coqui TTS, Windows SAPI, edge-tts)
- Implement polling-based or WebSocket communication
- Expected latency increase: 2-5 seconds

### 2. Function Calling Format Differences

**Current Gemini Format (used throughout codebase):**
```python
TOOL_DECLARATIONS = [
    {
        "name": "open_app",
        "description": "Opens any application...",
        "parameters": {
            "type": "OBJECT",
            "properties": {
                "app_name": {
                    "type": "STRING",
                    "description": "Exact name of the application"
                }
            },
            "required": ["app_name"]
        }
    }
]
```

**Ollama/OpenAI Compatible Format:**
```python
tools = [{
    "type": "function",
    "function": {
        "name": "open_app",
        "description": "Opens any application...",
        "parameters": {
            "type": "object",
            "properties": {...},
            "required": ["app_name"]
        }
    }
}]
```

**Impact:** All tool declarations in `main.py` (lines 140-473) need restructuring.

### 3. Vision Processing (Screen/Camera)

**Current Implementation:**
```python
# actions/screen_processor.py
async with client.aio.live.connect(model=LIVE_MODEL, config=config) as session:
    # Streams image + receives audio response directly
```

**Ollama Alternative:**
- Ollama supports vision with compatible models (e.g., Llava, BakLLaVa)
- However, requires sending base64-encoded images
- Audio response must be handled separately via TTS
- **File:** `actions/screen_processor.py` needs complete rewrite

### 4. Web Search Grounding

**Current Implementation:**
```python
# actions/web_search.py
client.models.generate_content(
    model="gemini-2.5-flash-lite",
    contents=query,
    config={"tools": [{"google_search": {}}]}  # Native Google Search
)
```

**Ollama Limitation:**
- No built-in web search capability
- Must rely on external search (DuckDuckGo already implemented as fallback)
- **Mitigation:** Already has DuckDuckGo fallback in `web_search.py`

### 5. Two Different Gemini SDKs

The codebase currently uses both:
- `google.genai` (new SDK) - for live streaming in `main.py` and `screen_processor.py`
- `google.generativeai` (old SDK) - for regular calls in `agent/*.py` and `actions/*.py`

**Impact:** All imports and client initialization need replacement.

---

## Cloud Models vs Ollama

### Important Distinction

**Kimi k2.5** (by Moonshot AI) is **NOT** available through Ollama—it is a **cloud API model** similar to GPT-4 or Claude.

**Ollama** is primarily for **locally run models**:
- Llama 3.x
- Mistral
- CodeLlama
- Phi-3
- Gemma
- Etc.

To use Kimi k2.5, you would need to:
1. Use Moonshot AI API directly
2. Use an OpenAI-compatible proxy

---

## Functional Parity Comparison

| Feature | Gemini API | Ollama (Local) | Notes |
|---------|------------|----------------|-------|
| Text generation | ✅ | ✅ | Works similarly |
| Function calling | ✅ | ✅ | Different format required |
| Vision (images) | ✅ | ✅* | Requires vision-capable model (Llava, etc.) |
| Audio streaming | ✅ Native | ❌ | Major architectural difference |
| Web search grounding | ✅ Built-in | ❌ | Requires external search API |
| Speech-to-Text | ✅ Native | ❌ | Requires external STT service |
| Text-to-Speech | ✅ Native | ❌ | Requires external TTS service |
| Response speed | Cloud latency | Local = faster | Ollama can be faster for text |
| Cost | Per-token | Free (local GPU) | Ollama has no API costs |
| Privacy | Data to Google | Local only | Ollama keeps data private |

---

## Files Requiring Modification

### High Complexity (Architecture Changes)
| File | Lines | Changes Required |
|------|-------|------------------|
| `main.py` | 853 | Replace Live API with STT+LLM+TTS pipeline |
| `actions/screen_processor.py` | 404 | Replace vision+audio streaming with image+TTS |

### Medium Complexity (SDK Swap)
| File | Lines | Changes Required |
|------|-------|------------------|
| `agent/planner.py` | 258 | Replace `google.generativeai` with Ollama client |
| `agent/executor.py` | 398 | Replace `google.generativeai` with Ollama client |
| `agent/error_handler.py` | 198 | Replace `google.generativeai` with Ollama client |
| `actions/web_search.py` | 135 | Remove Gemini search, keep DDG fallback |

### Low Complexity (Text Generation)
| File | Changes Required |
|------|------------------|
| `actions/code_helper.py` | Replace LLM calls |
| `actions/dev_agent.py` | Replace LLM calls |
| `actions/youtube_video.py` | Replace LLM calls |
| `actions/flight_finder.py` | Replace LLM calls |
| `actions/browser_control.py` | Replace LLM calls |
| `actions/file_controller.py` | Replace LLM calls |
| `actions/cmd_control.py` | Replace LLM calls |
| `actions/desktop.py` | Replace LLM calls |
| `actions/computer_settings.py` | Replace LLM calls |
| `actions/computer_control.py` | Replace LLM calls |
| `memory/config_manager.py` | Update config key name |

**Total Files Affected:** ~23 files

---

## Migration Options

### Option 1: Full Ollama Migration
**Approach:** Replace all Gemini usage with Ollama + external services

**Pros:**
- Complete privacy (all local)
- No API costs
- Works offline

**Cons:**
- Requires STT + TTS implementation (~2-5s latency)
- No native audio streaming
- Significant development effort (estimated 2-3 weeks)
- Requires capable local GPU for good performance

**Recommended Architecture:**
```
Microphone → [STT: Whisper/Vosk] → Text → [Ollama] → Text → [TTS: pyttsx3/Coqui] → Speaker
                                    ↓
                              [Function Calls]
```

### Option 2: Hybrid Approach (Recommended)
**Approach:** Keep Gemini for live audio, use Ollama for non-audio tasks

**Pros:**
- Minimal changes to core experience
- Reduces API costs for text-heavy operations
- Can fallback to local if internet is down

**Cons:**
- Still requires Gemini API key
- Two different LLM providers to maintain

**Implementation:**
```python
# Add LLM abstraction layer
def get_llm_client(mode="auto"):
    if mode == "audio" or not OLLAMA_AVAILABLE:
        return GeminiClient()
    return OllamaClient()
```

### Option 3: Use Kimi k2.5 via Moonshot AI API
**Approach:** Replace Gemini with Moonshot AI API (OpenAI-compatible)

**Pros:**
- Kimi k2.5 is competitive with Gemini
- Simple SDK swap (OpenAI-compatible)
- Can use with existing architecture (no STT/TTS needed if using Kimi's multimodal API)

**Cons:**
- Still cloud-based (not local)
- Requires Moonshot API key
- May not support audio streaming like Gemini Live

**Implementation:**
```python
from openai import OpenAI

client = OpenAI(
    api_key="moonshot-api-key",
    base_url="https://api.moonshot.cn/v1"
)
```

---

## Recommended Ollama Models

If proceeding with Ollama, these models offer best compatibility:

| Use Case | Recommended Model | Size | Notes |
|----------|------------------|------|-------|
| General tasks | Llama 3.1 | 8B-70B | Good all-rounder |
| Coding | CodeLlama | 7B-34B | Optimized for code generation |
| Vision | Llava 1.5/1.6 | 7B-13B | Image understanding |
| Fast responses | Phi-3 Mini/Medium | 3.8B-14B | Lightweight, fast |
| Function calling | Mistral / Mixtral | 7B-8x7B | Good tool use |

---

## Required Dependencies Changes

### Current requirements.txt
```
google-genai
google-generativeai
```

### Ollama requirements.txt
```
ollama              # Ollama Python client
openai              # For OpenAI-compatible API
# STT Options:
openai-whisper      # OpenAI Whisper for STT
# or
vosk                # Offline STT
# TTS Options:
pyttsx3             # Windows built-in TTS
# or
coqui-tts           # High quality neural TTS
edge-tts            # Microsoft Edge TTS (online)
```

---

## Estimation of Work

| Task | Estimated Time | Complexity |
|------|---------------|------------|
| Create LLM abstraction layer | 4-6 hours | Medium |
| Modify `main.py` for STT+TTS | 16-24 hours | High |
| Modify `screen_processor.py` | 8-12 hours | High |
| Update `agent/*.py` files | 4-6 hours | Medium |
| Update `actions/*.py` files | 8-12 hours | Medium |
| Testing and debugging | 8-16 hours | Medium |
| **Total** | **48-76 hours** | **2-3 weeks** |

---

## Conclusion

Migrating from Gemini to Ollama is **technically feasible** but requires significant architectural changes, primarily due to the lack of native audio streaming in Ollama.

**Recommendation:**
1. **For privacy-focused users:** Implement Option 2 (Hybrid) to gradually migrate
2. **For cost savings:** Consider using Kimi k2.5 via Moonshot API instead (easier migration)
3. **For offline capability:** Proceed with full Ollama migration, but expect latency trade-offs

---

## Next Steps

1. **Decision:** Choose migration option based on priorities (privacy/cost/latency)
2. **Prototype:** Test Ollama with a single action file first
3. **STT/TTS Selection:** Evaluate Whisper vs Vosk for STT, Coqui vs pyttsx3 for TTS
4. **Model Selection:** Download and test recommended Ollama models
5. **Implementation:** Create LLM abstraction layer before modifying individual files

---

*Document generated: 2026-03-06*
*Analyzed codebase: MARK XXV JARVIS Assistant*
