# Quick Task 54: Summary (Updated)

## Status: Done

## Probleem
Discord #algemeen geeft herhaaldelijk "request timed out before a response was generated".

## Root Causes (meerdere lagen)
1. **Dag 1**: `minimax-portal` provider gebruikte verlopen OAuth token (`minimax-oauth`) → gefixt met directe API key
2. **Dag 2**: Config verdween na nightly update → hersteld uit backup
3. **Dag 2**: MetaClaw proxy routeerde naar verkeerd MiniMax endpoint (`api.minimax.chat` ipv `api.minimax.io/anthropic`) → 401 → timeout
4. **Dag 2**: MetaClaw kan alleen OpenAI-format (`/chat/completions`), maar de MiniMax API key werkt alleen op het Anthropic-format endpoint (`api.minimax.io/anthropic/v1/messages`)

## Definitieve Fix
- MetaClaw proxy uitgeschakeld (incompatibel met Anthropic-only MiniMax key)
- Agent model direct gerouteerd naar `minimax-portal/MiniMax-M2.7`
- MetaClaw provider verwijderd uit openclaw.json
- Update script verbeterd: config backup/restore bij missing config

## MetaClaw Status
MetaClaw kan pas weer gebruikt worden als:
1. Een MiniMax OpenAI-format API key beschikbaar is (`api.minimax.chat`), OF
2. MetaClaw support krijgt voor Anthropic-format endpoints
