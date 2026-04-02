---
type: quick
number: 54
description: "Fix structurele Discord request timed out errors: MiniMax provider timeout, failover/retry, MetaClaw proxy latency"
date: 2026-04-02
---

# Quick Task 54: Fix Discord Timeout Errors

## Probleem
Discord #algemeen geeft herhaaldelijk "request timed out before a response was generated".
Logs tonen: `Profile minimax-portal:default timed out. Trying next account...` → `decision: surface_error`.

## Root Cause
1. MiniMax API calls duren soms langer dan het provider timeout (default: 60s?)
2. Geen failover naar ander model wanneer MiniMax timeout
3. MetaClaw proxy (poort 30000) voegt extra latency toe aan de keten
4. OpenClaw surfacet de error ipv retry

## Taken

### Task 1: Diagnose exacte timeout keten
- Check MetaClaw proxy latency (direct MiniMax vs via MetaClaw)
- Check OpenClaw provider timeout config
- Check Discord interaction timeout (3s acknowledge, 15min followup)

### Task 2: Configureer MiniMax provider timeout
- Stel hogere timeout in voor MiniMax provider in openclaw.json
- Configureer retry bij timeout

### Task 3: Configureer failover model
- Voeg fallback provider toe wanneer MiniMax faalt
- Zorg dat failover automatisch werkt, niet surface_error
