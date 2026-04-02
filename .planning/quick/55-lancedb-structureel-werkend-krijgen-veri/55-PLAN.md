---
type: quick
number: 55
description: "LanceDB structureel werkend krijgen: verify patches, auto-capture, embedding model"
date: 2026-04-02
---

# Quick Task 55: LanceDB Structureel Werkend

## Diagnose resultaat
- LanceDB plugin laadt en werkt (1 memory, auto-capture actief)
- `@lancedb/lancedb` npm package: geïnstalleerd ✓
- `dist/package.json`: aanwezig ✓
- Hardlinks: gebroken ✓
- `DEFAULT_CAPTURE_MAX_CHARS`: al 2000 (v2026.4.1 default of config override) ✓
- Embedding model: `text-embedding-3-small` (OpenAI) ✓
- `captureMaxChars: 2000` in plugin config ✓
- **MEMORY_TRIGGERS**: Tsjechisch + Engels, GEEN Nederlands ✗
- **shouldCapture markdown rejection**: niet meer aanwezig in v2026.4.1 (opgelost upstream) ✓
- **autoCapture text-cleaning**: niet gepatcht (check of nog nodig) ?

## Taken

### Task 1: Patch MEMORY_TRIGGERS met Nederlandse regexes
De Dutch triggers zijn niet meer in index.js na de v2026.4.1 update.
Voeg Nederlandse regexes toe naast de bestaande Tsjechische/Engelse.

### Task 2: Verify autoCapture text-cleaning status
Check of v2026.4.1 de text-cleaning fix al bevat of dat de patch nog nodig is.

### Task 3: Fix update script patching voor v2026.4.1
De update script patches mislukten stilletjes. Verify en fix de patch logica.
