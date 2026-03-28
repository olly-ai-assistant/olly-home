---
type: quick
number: 53
description: "Fix discord-voice-ai audio receive: voice_recv DAVE decryptie + Gemini Live end-to-end"
date: 2026-03-28
must_haves:
  truths:
    - "Bot ontvangt decoded PCM audio van Discord gebruikers (Sink.write wordt aangeroepen)"
    - "Audio wordt doorgestuurd naar Gemini Live en response audio wordt afgespeeld"
    - "Playback is smooth (geen horten en stoten)"
  artifacts:
    - "/home/olly/discord-voice-ai/bot.py"
    - "/home/olly/discord-voice-ai/gemini_live.py"
  key_links:
    - "GitHub issue: imayhaveborkedit/discord-ext-voice-recv#53"
    - "PR met fix: vocolboy/discord-ext-voice-recv PR#56"
---

# Quick Task 53: Fix Discord Voice AI Audio Receive + Gemini Live

## Probleem
De discord-voice-ai bot kan verbinden met Discord voice channels en audio AFSPELEN (greeting werkt),
maar kan GEEN audio ONTVANGEN van gebruikers. voice_recv's Sink.write() wordt nooit aangeroepen.

## Root Cause Analyse (tot nu toe)
1. Discord voice protocol vereist nu DAVE (end-to-end encryption) — discord.py 2.7 + davey 0.1.4
2. voice_recv v0.5.3a180 (PyPI) ondersteunt DAVE niet → CryptoError
3. voice_recv v0.5.3a185 (vocolboy fork, PR#56) heeft DAVE support, maar:
   - UDP packets komen WEL binnen (bewezen via iptables counter: 10 pkt/5s)
   - RAW socket listener ontvangt 52-byte RTCP keepalives
   - Maar GEEN audio RTP packets bereiken voice_recv's callback
4. VPN (wg0) moet UIT zijn voor voice UDP
5. UFW INPUT policy DROP moet UDP ACCEPT hebben

## Huidige Setup
- discord.py 2.7.1
- davey 0.1.4
- voice_recv 0.5.3a185 (pip install from git+https://github.com/vocolboy/discord-ext-voice-recv.git@main)
- Gemini Live API: gemini-3.1-flash-live-preview (getest, werkt)
- GEMINI_API_KEY: geconfigureerd in .env
- PIPELINE_MODE: gemini-live

## Taken

### Task 1: Debug waarom voice_recv geen audio RTP ontvangt
- Enable DEBUG logging voor voice_recv reader
- Check of de reader thread actief is
- Vergelijk socket binding (bot's UDP socket vs waar packets aankomen)
- Check of DAVE protocol handshake correct afgerond wordt
- Mogelijk: test met voice_recv's eigen example/recv.py

### Task 2: Fix audio receive
- Op basis van debug info: patch voice_recv of de bot code
- Verifieer dat Sink.write() aangeroepen wordt met PCM data
- Verifieer dat audio_queue gevuld wordt

### Task 3: Fix Gemini Live playback (choppy → smooth)
- Buffer audio chunks voordat ze afgespeeld worden
- Speel als continue PCM stream af, niet per chunk
- Test latentie

### Task 4: End-to-end test
- Gebruiker spreekt → bot verstaat → Gemini antwoordt → bot speelt audio af
- Verifieer rondtriptijd
- Commit en push

## Vereisten voor Voice
```bash
# VPN uit
sudo wg-quick down wg0

# Firewall UDP open
sudo iptables -I INPUT -p udp -j ACCEPT

# DNS op publieke resolver
echo -e "nameserver 1.1.1.1\nnameserver 8.8.8.8" | sudo tee /etc/resolv.conf
```
