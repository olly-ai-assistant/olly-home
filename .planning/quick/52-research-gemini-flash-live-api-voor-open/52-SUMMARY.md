# Research: Gemini Flash Live API voor OpenClaw

## Wat is Gemini Live API?

Real-time, bidirectionele **audio/video/tekst streaming** via WebSocket (WSS). Ontworpen voor spraakassistenten met lage latentie. Vergelijkbaar met OpenAI Realtime API.

**Model:** `gemini-live-2.5-flash-preview` (nieuwste) of `gemini-2.0-flash-live-preview-04-09`

## Technische Specificaties

| Eigenschap | Waarde |
|------------|--------|
| Protocol | WebSocket (WSS) |
| Audio input | 16-bit PCM, 16kHz, mono, little-endian |
| Audio output | 16-bit PCM, 24kHz, mono, little-endian |
| Video input | JPEG frames, max 1 FPS |
| Talen | 70+ talen |
| Barge-in | Ja (interrupt mid-response) |
| Tool calling | Ja (function calling + Google Search) |
| Context | Stateful sessie (multi-turn) |

## SDK Support

| Platform | SDK | Status |
|----------|-----|--------|
| **Python** | `google-genai` | Volledig ondersteund |
| **JavaScript/Node.js** | `@google/genai` | Volledig ondersteund |
| **WebSocket direct** | Standaard WSS | Werkt met elke taal |

### Python Voorbeeld (Server-side)
```python
from google import genai
from google.genai import types

client = genai.Client(api_key="GEMINI_API_KEY")
session = await client.aio.live.connect(
    model="gemini-live-2.5-flash-preview",
    config=types.LiveConnectConfig(
        response_modalities=[types.Modality.AUDIO],
    )
)
# Stuur audio
session.send_realtime_input(audio=audio_blob)
# Ontvang audio
async for msg in session:
    if msg.server_content and msg.server_content.audio:
        play(msg.server_content.audio.data)
```

### JavaScript Voorbeeld
```javascript
const session = await ai.live.connect({
    model: 'gemini-live-2.5-flash-preview',
    config: { responseModalities: [Modality.AUDIO] },
    callbacks: {
        onmessage: (e) => playAudio(e.data),
    }
});
session.sendRealtimeInput({ audio: pcmBlob });
```

## Pricing

| Component | Prijs |
|-----------|-------|
| **Audio input** | $0.005/minuut of $3.00/1M tokens |
| **Audio output** | $0.018/minuut of $12.00/1M tokens |
| **Tekst input** | $0.75/1M tokens |
| **Tekst output** | $4.50/1M tokens |
| **Free tier** | Ja, met limieten |

### Vergelijking met MiniMax M2.7

| | Gemini Live | MiniMax M2.7 |
|---|---|---|
| Real-time audio | Ja (native) | Nee (tekst-only) |
| Latentie | ~200-500ms | ~1-3s |
| Pricing | $0.023/min (in+out) | Inclusief in abonnement |
| Voice quality | Google WaveNet | N.v.t. (gebruikt Edge TTS) |
| Talen | 70+ | Onbeperkt (tekst) |
| Tool calling | Ja | Ja |

## Integratie Mogelijkheden

### 1. Discord Voice Channels ✅ MOGELIJK

De `discord-voice-ai` bot gebruikt al Discord's voice infra (PCM audio frames via `discord.py`). Integratie:

**Architectuur:**
```
Discord Voice → PCM 48kHz → Resample 16kHz → Gemini Live WS → PCM 24kHz → Resample 48kHz → Discord Voice
```

**Voordelen vs huidige setup (Whisper + Claude + Edge TTS):**
- 1 API call ipv 3 (STT + LLM + TTS)
- Veel lagere latentie (~300ms vs ~3-5s)
- Native barge-in (interrupt mid-response)
- Betere conversational flow

**Implementatie:**
- Vervang `pipeline.py` STT→LLM→TTS keten door één Gemini Live sessie
- Resample audio (48kHz↔16kHz/24kHz)
- Houd Edge TTS als fallback voor tekst-only antwoorden

**Geschatte moeite:** Medium (1-2 dagen). De audio pipeline structuur bestaat al.

### 2. Telegram Voice Messages ✅ MOGELIJK

OpenClaw kan al voice messages transcriberen (via Groq Whisper). Met Gemini Live:

**Optie A: Voice-to-voice (Live API)**
- Ontvang voice message → stuur audio naar Gemini Live → ontvang audio antwoord → stuur als voice message terug
- Pro: Natuurlijke voice-to-voice ervaring
- Con: Telegram voice messages zijn niet real-time (het is record→send→receive→play)

**Optie B: Voice-to-text-to-voice**
- Transcribeer voice message → stuur tekst naar Gemini → ontvang tekst → TTS terug
- Pro: Goedkoper, werkt met elk model
- Con: Langzamer, geen native voice

**Aanbeveling:** Optie A is indrukwekkender maar Telegram's asynchrone model (geen live streaming) maakt de Live API minder nuttig. De huidige Groq Whisper + MiniMax + Edge TTS pipeline is effectiever voor Telegram.

### 3. OpenClaw als Gemini Live Provider ⚠️ COMPLEX

OpenClaw zou Gemini Live als model provider kunnen ondersteunen:
- Nieuwe provider in `models.providers.gemini-live`
- WebSocket sessie per conversatie
- Audio passthrough voor Discord/Telegram voice

**Maar:** OpenClaw's architectuur is request/response-gebaseerd, niet streaming. Een Live API provider zou significant refactoring vereisen van de session runtime.

### 4. Situational: Gemini als Voice Agent via Discord ✅ BEST FIT

De beste use case: **vervang de huidige discord-voice-ai pipeline door Gemini Live**.

| Stap | Huidig | Met Gemini Live |
|------|--------|-----------------|
| 1. Spraak ontvangen | Discord → PCM | Discord → PCM (zelfde) |
| 2. Transcriberen | Whisper API (~1s) | ⏭️ (niet nodig) |
| 3. LLM verwerken | Claude haiku (~1s) | ⏭️ (gecombineerd) |
| 4. TTS genereren | Edge TTS (~1s) | ⏭️ (gecombineerd) |
| 5. Audio terugsturen | PCM → Discord | PCM → Discord (zelfde) |
| **Totaal** | **~3-5s** | **~300ms** |

## Aanbevelingen

### Kort termijn (quick win)
1. **Maak een Gemini API key aan** op [aistudio.google.com](https://aistudio.google.com)
2. **Experimenteer** met de Live API via Python SDK
3. **Prototype** in discord-voice-ai met Gemini Live als alternatieve pipeline

### Medium termijn
4. **Dual-mode** in discord-voice-ai: Gemini Live voor voice, Claude voor tool queries
5. **Fallback**: als Gemini Live faalt → huidige Whisper+Claude+TTS pipeline

### Niet aanbevolen
- OpenClaw core omschrijven naar streaming (te veel werk, te weinig winst)
- Telegram voice-to-voice via Live API (Telegram's model is niet real-time)

## Conclusie

Gemini Live API is **ideaal voor Discord voice** — het vervangt 3 API calls door 1, verlaagt latentie van 3-5s naar ~300ms, en ondersteunt barge-in. De beste integratie is in `discord-voice-ai`, niet in OpenClaw zelf.

Voor Telegram blijft de huidige setup (Groq transcriptie + MiniMax tekst + optioneel TTS) de beste keuze.
