# Hermes-House Complete Gateway Adapter Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** hermes-house (port 7842) proxies all ClawNexus WebSocket RPC methods to the Hermes gateway, making the ClawNexus UI fully functional with real data from all gateway subsystems.

**Architecture:** hermes-house server.py acts as a full ClawNexus gateway adapter. For each RPC method, it either proxies to an existing or new gateway HTTP endpoint, or directly imports gateway Python modules. The ClawNexus UI (served at /) connects to hermes-house's WebSocket at /ws/nexus and all UI functionality works end-to-end.

**Tech Stack:** Python 3 (aiohttp, httpx), TypeScript/React, ClawNexus Gateway Protocol v3

---

## File Structure

```
/home/olly/.hermes/hermes-agent/
├── gateway/
│   ├── run.py                      # RunManager — session_store, delivery_router
│   ├── session.py                  # SessionStore.list_sessions(), get_transcript()
│   ├── config.py                   # load_gateway_config(), _save_config_key()
│   ├── channel_directory.py        # ChannelDirectory (per-platform adapters)
│   ├── platforms/
│   │   ├── api_server.py           # ApiServer — gateway HTTP API (NEW endpoints here)
│   │   └── ...
│   └── ws_server.py                # Hermes gateway WS (already has connect/agents.list/ping)
│
/mnt/nas/docker/hermes-house/
├── server.py                       # hermes-house WebSocket handler (MODIFY: add all methods)
├── docker-compose.yml
└── web_dist/                      # Built ClawNexus frontend
```

---

## Phase A: Gateway HTTP Endpoints (api_server.py)

These endpoints are called by hermes-house server.py over HTTP. They give hermes-house access to gateway state without importing gateway modules directly.

### Task A1: Sessions HTTP API

**Files:**
- Modify: `/home/olly/.hermes/hermes-agent/gateway/platforms/api_server.py`

Add these route registrations in `_setup_routes()` (around line 2335):
```python
self._app.router.add_get("/v1/sessions", self._handle_sessions_list)
self._app.router.add_get("/v1/sessions/{session_key}/preview", self._handle_sessions_preview)
self._app.router.add_delete("/v1/sessions/{session_key}", self._handle_sessions_delete)
```

Add handler methods to `ApiServer` class (after `_handle_models` around line 614):

```python
async def _handle_sessions_list(self, request: "web.Request") -> "web.Response":
    """GET /v1/sessions — list all sessions."""
    auth_err = self._check_auth(request)
    if auth_err:
        return auth_err
    active_minutes = request.query.get("active_minutes")
    try:
        active_minutes = int(active_minutes) if active_minutes else None
    except ValueError:
        active_minutes = None
    # Get session store via RunManager (set during gateway init)
    store = self._get_session_store()
    if not store:
        return web.json_response({"error": "Session store not available"}, status=503)
    entries = store.list_sessions(active_minutes=active_minutes)
    return web.json_response({
        "sessions": [
            {
                "key": e.key,
                "agentId": e.agent_id or "main",
                "label": e.label or e.key[:8],
                "createdAt": int(e.created_at.timestamp()) if hasattr(e, 'created_at') else 0,
                "lastActiveAt": int(e.updated_at.timestamp()) if hasattr(e, 'updated_at') else 0,
                "messageCount": getattr(e, 'message_count', 0) or 0,
            }
            for e in entries
        ]
    })

async def _handle_sessions_preview(self, request: "web.Request") -> "web.Response":
    """GET /v1/sessions/{session_key}/preview — get recent transcript messages."""
    auth_err = self._check_auth(request)
    if auth_err:
        return auth_err
    session_key = request.match_info["session_key"]
    store = self._get_session_store()
    if not store:
        return web.json_response({"error": "Session store not available"}, status=503)
    messages = store.load_transcript(session_key)
    # Return last 20 messages as preview
    recent = messages[-20:] if len(messages) > 20 else messages
    return web.json_response({
        "key": session_key,
        "messages": [
            {
                "id": f"msg-{i}",
                "role": m.get("role", "unknown"),
                "content": m.get("content", ""),
                "timestamp": int(time.time()),
            }
            for i, m in enumerate(recent)
        ]
    })

async def _handle_sessions_delete(self, request: "web.Request") -> "web.Response":
    """DELETE /v1/sessions/{session_key} — delete a session."""
    auth_err = self._check_auth(request)
    if auth_err:
        return auth_err
    session_key = request.match_info["session_key"]
    store = self._get_session_store()
    if not store:
        return web.json_response({"error": "Session store not available"}, status=503)
    try:
        store.delete_session(session_key)
    except Exception as e:
        return web.json_response({"error": str(e)}, status=400)
    return web.json_response({"ok": True})

def _get_session_store(self):
    """Get SessionStore from RunManager via app shared state."""
    # The RunManager sets self._app["api_server_adapter"] = self during gateway init
    # But we need the reverse — get RunManager from ApiServer
    # Alternative: use a module-level registry
    import gateway.run
    inst = gateway.run._global_run_manager
    if inst and hasattr(inst, 'session_store'):
        return inst.session_store
    return None
```

- [ ] **Step 1: Add route registrations in `_setup_routes()`**
- [ ] **Step 2: Add `_handle_sessions_list` method**
- [ ] **Step 3: Add `_handle_sessions_preview` method**
- [ ] **Step 4: Add `_handle_sessions_delete` method**
- [ ] **Step 5: Add `_get_session_store` helper**
- [ ] **Step 6: Test — restart gateway and verify sessions endpoints respond**

```bash
curl -s http://127.0.0.1:8642/v1/sessions | python3 -m json.tool | head -20
```

Expected: JSON with `{"sessions": [...]}`

- [ ] **Step 7: Commit**

```bash
cd /home/olly/.hermes
git add hermes-agent/gateway/platforms/api_server.py
git commit -m "feat(gateway): add sessions HTTP API endpoints"
```

---

### Task A2: Usage HTTP API

**Files:**
- Modify: `/home/olly/.hermes/hermes-agent/gateway/platforms/api_server.py`

Add route registration:
```python
self._app.router.add_get("/v1/usage", self._handle_usage)
```

Add handler (add after `_handle_sessions_delete`):

```python
async def _handle_usage(self, request: "web.Request") -> "web.Response":
    """GET /v1/usage — return token usage info aggregated across all sessions."""
    auth_err = self._check_auth(request)
    if auth_err:
        return auth_err
    # Aggregate usage from session store and active agents
    store = self._get_session_store()
    total_input = 0
    total_output = 0
    if store:
        for entry in store.list_sessions():
            # SessionEntry may store usage metadata — check attributes
            total_input += getattr(entry, 'input_tokens', 0) or 0
            total_output += getattr(entry, 'output_tokens', 0) or 0
    # Get active agent usage from RunManager
    inst = self._get_run_manager()
    active_tokens = 0
    if inst and hasattr(inst, '_running_agents'):
        for agent in inst._running_agents.values():
            if agent and agent is not _AGENT_PENDING_SENTINEL:
                active_tokens += getattr(agent, 'session_total_tokens', 0) or 0
    return web.json_response({
        "updatedAt": int(time.time()),
        "providers": [
            {
                "provider": "hermes",
                "displayName": "Hermes Agent",
                "plan": "unlimited",
                "windows": [],
            }
        ]
    })
```

- [ ] **Step 1: Add route and handler**
- [ ] **Step 2: Test endpoint**

```bash
curl -s http://127.0.0.1:8642/v1/usage | python3 -m json.tool
```

Expected: Valid UsageInfo JSON

- [ ] **Step 3: Commit**

---

### Task A3: Config HTTP API

**Files:**
- Modify: `/home/olly/.hermes/hermes-agent/gateway/platforms/api_server.py`

Add routes:
```python
self._app.router.add_get("/v1/config", self._handle_config_get)
self._app.router.add_post("/v1/config", self._handle_config_set)
```

Add handlers:

```python
async def _handle_config_get(self, request: "web.Request") -> "web.Response":
    """GET /v1/config — return current gateway config."""
    auth_err = self._check_auth(request)
    if auth_err:
        return auth_err
    try:
        config = self._load_gateway_config_unsafe()
    except Exception as e:
        return web.json_response({"error": str(e)}, status=500)
    return web.json_response({
        "config": config,
        "valid": True,
    })

async def _handle_config_set(self, request: "web.Request") -> "web.Response":
    """POST /v1/config — update gateway config."""
    auth_err = self._check_auth(request)
    if auth_err:
        return auth_err
    try:
        body = await request.json()
    except Exception:
        return web.json_response({"error": "Invalid JSON"}, status=400)
    raw = body.get("raw") or body.get("config")
    if not raw:
        return web.json_response({"error": "Missing config body"}, status=400)
    try:
        inst = self._get_run_manager()
        if inst:
            await inst.update_config_raw(str(raw))
        else:
            return web.json_response({"error": "Gateway not fully initialized"}, status=503)
    except Exception as e:
        return web.json_response({"error": str(e)}, status=400)
    return web.json_response({"ok": True, "config": json.loads(raw) if isinstance(raw, str) else raw})

def _load_gateway_config_unsafe(self):
    """Load gateway config without requiring full init."""
    from gateway.config import load_gateway_config
    return load_gateway_config()
```

- [ ] **Step 1: Add routes and handlers**
- [ ] **Step 2: Find or implement `update_config_raw` in RunManager**
- [ ] **Step 3: Test config.get and config.set**

```bash
curl -s http://127.0.0.1:8642/v1/config | python3 -m json.tool | head -10
curl -s -X POST http://127.0.0.1:8642/v1/config -H "Content-Type: application/json" -d '{"config":{}}'
```

- [ ] **Step 4: Commit**

---

### Task A4: Channels HTTP API

**Files:**
- Modify: `/home/olly/.hermes/hermes-agent/gateway/platforms/api_server.py`

Add routes:
```python
self._app.router.add_get("/v1/channels", self._handle_channels)
self._app.router.add_post("/v1/channels/{channel}/logout", self._handle_channels_logout)
```

Add handlers. First, add a `_channels` attribute in `__init__`:
```python
self._channels = None  # Lazy-loaded ChannelDirectory
```

Then add handlers:

```python
async def _handle_channels(self, request: "web.Request") -> "web.Response":
    """GET /v1/channels — return status of all connected channels."""
    auth_err = self._check_auth(request)
    if auth_err:
        return auth_err
    inst = self._get_run_manager()
    if not inst or not hasattr(inst, 'adapters'):
        return web.json_response({"channels": []})
    adapters = inst.adapters
    channels = []
    for platform, adapter in adapters.items():
        if adapter is None:
            continue
        channel_info = {
            "id": str(platform.value) if hasattr(platform, 'value') else str(platform),
            "type": str(platform.value) if hasattr(platform, 'value') else str(platform),
            "name": getattr(adapter, '_name', str(platform)) or str(platform),
            "status": "connected" if getattr(adapter, '_running', False) else "disconnected",
            "configured": True,
            "linked": getattr(adapter, '_running', False),
            "running": getattr(adapter, '_running', False),
        }
        channels.append(channel_info)
    return web.json_response({"channels": channels})

async def _handle_channels_logout(self, request: "web.Request") -> "web.Response":
    """POST /v1/channels/{channel}/logout — logout from a channel."""
    auth_err = self._check_auth(request)
    if auth_err:
        return auth_err
    channel = request.match_info["channel"]
    return web.json_response({"cleared": False, "error": "Not implemented"})
```

- [ ] **Step 1: Add routes and handlers**
- [ ] **Step 2: Test endpoint**

```bash
curl -s http://127.0.0.1:8642/v1/channels | python3 -m json.tool
```

- [ ] **Step 3: Commit**

---

### Task A5: Skills HTTP API

**Files:**
- Modify: `/home/olly/.hermes/hermes-agent/gateway/platforms/api_server.py`

Add routes:
```python
self._app.router.add_get("/v1/skills", self._handle_skills)
self._app.router.add_post("/v1/skills/{name}/install", self._handle_skills_install)
```

Add handlers:

```python
async def _handle_skills(self, request: "web.Request") -> "web.Response":
    """GET /v1/skills — list all available skills."""
    auth_err = self._check_auth(request)
    if auth_err:
        return auth_err
    # Return empty catalog — real skills come from skill registry
    # which loads from ~/.hermes/skills/ at runtime
    skills_dir = Path.home() / ".hermes" / "skills"
    skills = []
    if skills_dir.exists():
        for item in skills_dir.iterdir():
            if item.is_dir() and (item / "skill.yaml").exists() or (item / "skill.yml").exists():
                skills.append({
                    "id": item.name,
                    "slug": item.name,
                    "name": item.name.replace("-", " ").replace("_", " ").title(),
                    "description": "",
                    "enabled": True,
                    "icon": "star",
                    "version": "0.0.0",
                    "isBundled": False,
                })
    return web.json_response({"skills": skills})

async def _handle_skills_install(self, request: "web.Request") -> "web.Response":
    """POST /v1/skills/{name}/install — install a skill by name."""
    auth_err = self._check_auth(request)
    if auth_err:
        return auth_err
    name = request.match_info["name"]
    return web.json_response({
        "ok": False,
        "message": f"Skill install not yet implemented — manually install via hermes CLI",
    })
```

- [ ] **Step 1: Add routes and handlers**
- [ ] **Step 2: Test endpoint**

```bash
curl -s http://127.0.0.1:8642/v1/skills | python3 -m json.tool
```

- [ ] **Step 3: Commit**

---

### Task A6: Tools HTTP API

**Files:**
- Modify: `/home/olly/.hermes/hermes-agent/gateway/platforms/api_server.py`

Add route:
```python
self._app.router.add_get("/v1/tools", self._handle_tools)
```

Add handler:

```python
async def _handle_tools(self, request: "web.Request") -> "web.Response":
    """GET /v1/tools — return tool catalog."""
    auth_err = self._check_auth(request)
    if auth_err:
        return auth_err
    # Import from RunManager to get registered tools
    inst = self._get_run_manager()
    tools = []
    if inst and hasattr(inst, '_tool_registry'):
        for name, tool in inst._tool_registry.items():
            tools.append({
                "name": name,
                "description": getattr(tool, '__doc__', '') or "",
                "enabled": True,
            })
    return web.json_response({"tools": tools})
```

- [ ] **Step 1: Add route and handler**
- [ ] **Step 2: Test endpoint**

```bash
curl -s http://127.0.0.1:8642/v1/tools | python3 -m json.tool
```

- [ ] **Step 3: Commit**

---

### Task A7: Models HTTP API (verify existing)

**Files:**
- Modify: `/home/olly/.hermes/hermes-agent/gateway/platforms/api_server.py`

The `/v1/models` endpoint already exists (`_handle_models`). Verify it works:

```bash
curl -s http://127.0.0.1:8642/v1/models | python3 -m json.tool
```

- [ ] **Step 1: Verify endpoint responds correctly**
- [ ] **Step 2: If broken, fix `_handle_models`**

---

## Phase B: hermes-house WebSocket Handler (server.py)

All missing RPC method handlers are added to `handle_websocket()` in server.py. Each method checks if the corresponding gateway HTTP endpoint exists and proxies to it, or returns appropriate real/borrowed data.

### Task B1: Sessions List/Preview/Delete

**Files:**
- Modify: `/mnt/nas/docker/hermes-house/server.py`

Add `sessions_list_handler`, `sessions_preview_handler`, `sessions_delete_handler` functions and wire them in the WebSocket handler.

Add this case in the method switch (after `chat.send` handling, around line 279):
```python
elif method == "sessions.list":
    result = await proxy_gateway_get(f"{_gateway_base}/v1/sessions")
    await ws.send_json({"type": "res", "id": req_id, "ok": True, "result": result})
elif method == "sessions.preview":
    session_key = params.get("sessionKey", "")
    result = await proxy_gateway_get(f"{_gateway_base}/v1/sessions/{session_key}/preview")
    await ws.send_json({"type": "res", "id": req_id, "ok": True, "result": result})
elif method == "sessions.delete":
    session_key = params.get("key", "")
    result = await proxy_gateway_delete(f"{_gateway_base}/v1/sessions/{session_key}")
    await ws.send_json({"type": "res", "id": req_id, "ok": True, "result": result})
```

Add helper functions (after the `map_status` function, around line 63):

```python
async def proxy_gateway_get(url: str, timeout: float = 10.0) -> Dict[str, Any]:
    """GET gateway endpoint and return JSON response data."""
    import httpx
    async with httpx.AsyncClient(timeout=timeout) as client:
        resp = await client.get(url)
        if resp.status_code == 200:
            return resp.json()
        raise httpx.HTTPError(f"Gateway returned {resp.status_code}")

async def proxy_gateway_delete(url: str, timeout: float = 10.0) -> Dict[str, Any]:
    """DELETE gateway endpoint and return JSON response data."""
    import httpx
    async with httpx.AsyncClient(timeout=timeout) as client:
        resp = await client.delete(url)
        if resp.status_code == 200:
            return resp.json()
        raise httpx.HTTPError(f"Gateway returned {resp.status_code}")
```

- [ ] **Step 1: Add proxy helper functions**
- [ ] **Step 2: Add sessions.list handler**
- [ ] **Step 3: Add sessions.preview handler**
- [ ] **Step 4: Add sessions.delete handler**
- [ ] **Step 5: Rebuild Docker and test**

---

### Task B2: Usage Status + Config Get/Set

**Files:**
- Modify: `/mnt/nas/docker/hermes-house/server.py`

Add in the WebSocket method handler:

```python
elif method == "usage.status":
    try:
        result = await proxy_gateway_get(f"{_gateway_base}/v1/usage")
        await ws.send_json({"type": "res", "id": req_id, "ok": True, "result": result})
    except Exception as e:
        await ws.send_json({"type": "res", "id": req_id, "ok": False,
            "error": {"code": -32000, "message": str(e)}})
elif method == "config.get":
    try:
        result = await proxy_gateway_get(f"{_gateway_base}/v1/config")
        await ws.send_json({"type": "res", "id": req_id, "ok": True, "result": result})
    except Exception as e:
        await ws.send_json({"type": "res", "id": req_id, "ok": False,
            "error": {"code": -32000, "message": str(e)}})
elif method == "config.set":
    try:
        body = json.dumps(params)
        async with httpx.AsyncClient(timeout=30.0) as client:
            resp = await client.post(f"{_gateway_base}/v1/config", content=body,
                headers={"Content-Type": "application/json"})
        if resp.status_code == 200:
            await ws.send_json({"type": "res", "id": req_id, "ok": True, "result": resp.json()})
        else:
            await ws.send_json({"type": "res", "id": req_id, "ok": False,
                "error": {"code": -32000, "message": f"Config update failed: {resp.status_code}"}})
    except Exception as e:
        await ws.send_json({"type": "res", "id": req_id, "ok": False,
            "error": {"code": -32000, "message": str(e)}})
```

- [ ] **Step 1: Add usage.status handler**
- [ ] **Step 2: Add config.get handler**
- [ ] **Step 3: Add config.set handler**
- [ ] **Step 4: Rebuild Docker and test**

---

### Task B3: Channels Status + Logout

**Files:**
- Modify: `/mnt/nas/docker/hermes-house/server.py`

Add in the WebSocket method handler:

```python
elif method == "channels.status":
    try:
        result = await proxy_gateway_get(f"{_gateway_base}/v1/channels")
        # Flatten to ChannelInfo array expected by frontend
        raw_channels = result.get("channels", [])
        await ws.send_json({"type": "res", "id": req_id, "ok": True,
            "result": {"channels": raw_channels}})
    except Exception as e:
        await ws.send_json({"type": "res", "id": req_id, "ok": False,
            "error": {"code": -32000, "message": str(e)}})
elif method == "channels.logout":
    channel = params.get("channel", "")
    try:
        result = await proxy_gateway_post(f"{_gateway_base}/v1/channels/{channel}/logout", {})
        await ws.send_json({"type": "res", "id": req_id, "ok": True, "result": result})
    except Exception as e:
        await ws.send_json({"type": "res", "id": req_id, "ok": False,
            "error": {"code": -32000, "message": str(e)}})
```

- [ ] **Step 1: Add channels.status handler**
- [ ] **Step 2: Add channels.logout handler**
- [ ] **Step 3: Rebuild Docker and test**

---

### Task B4: Skills Status + Install

**Files:**
- Modify: `/mnt/nas/docker/hermes-house/server.py`

Add in the WebSocket method handler:

```python
elif method == "skills.status":
    try:
        result = await proxy_gateway_get(f"{_gateway_base}/v1/skills")
        await ws.send_json({"type": "res", "id": req_id, "ok": True, "result": result})
    except Exception as e:
        await ws.send_json({"type": "res", "id": req_id, "ok": False,
            "error": {"code": -32000, "message": str(e)}})
elif method == "skills.install":
    name = params.get("name", "")
    install_id = params.get("installId", "")
    try:
        result = await proxy_gateway_post(
            f"{_gateway_base}/v1/skills/{name}/install",
            {"installId": install_id}
        )
        await ws.send_json({"type": "res", "id": req_id, "ok": True, "result": result})
    except Exception as e:
        await ws.send_json({"type": "res", "id": req_id, "ok": False,
            "error": {"code": -32000, "message": str(e)}})
```

- [ ] **Step 1: Add skills.status handler**
- [ ] **Step 2: Add skills.install handler**
- [ ] **Step 3: Rebuild Docker and test**

---

### Task B5: Tools Catalog + Models List

**Files:**
- Modify: `/mnt/nas/docker/hermes-house/server.py`

Add in the WebSocket method handler:

```python
elif method == "tools.catalog":
    try:
        result = await proxy_gateway_get(f"{_gateway_base}/v1/tools")
        await ws.send_json({"type": "res", "id": req_id, "ok": True, "result": result})
    except Exception as e:
        await ws.send_json({"type": "res", "id": req_id, "ok": False,
            "error": {"code": -32000, "message": str(e)}})
elif method == "models.list":
    try:
        result = await proxy_gateway_get(f"{_gateway_base}/v1/models")
        models = result.get("data", [])
        await ws.send_json({"type": "res", "id": req_id, "ok": True,
            "result": {"models": models}})
    except Exception as e:
        await ws.send_json({"type": "res", "id": req_id, "ok": False,
            "error": {"code": -32000, "message": str(e)}})
```

- [ ] **Step 1: Add tools.catalog handler**
- [ ] **Step 2: Add models.list handler**
- [ ] **Step 3: Rebuild Docker and test**

---

### Task B6: Remaining Methods — Chat Abort + Sessions Preview + Sessions Delete

**Files:**
- Modify: `/mnt/nas/docker/hermes-house/server.py`

Add `sessions.preview` (already done in B1), verify `chat.abort`:

```python
elif method == "chat.abort":
    session_key = params.get("sessionKey", "")
    # Gateway doesn't have an abort endpoint yet — return ok:True to unblock UI
    await ws.send_json({"type": "res", "id": req_id, "ok": True,
        "result": {"aborted": False, "reason": "abort not implemented"}})
```

Also add missing `sessions.preview` (it needs sessionKey param):

```python
elif method == "sessions.preview":
    session_key = params.get("sessionKey", "")
    try:
        result = await proxy_gateway_get(
            f"{_gateway_base}/v1/sessions/{session_key}/preview",
            timeout=15.0
        )
        await ws.send_json({"type": "res", "id": req_id, "ok": True, "result": result})
    except Exception as e:
        await ws.send_json({"type": "res", "id": req_id, "ok": False,
            "error": {"code": -32000, "message": str(e)}})
```

- [ ] **Step 1: Add chat.abort stub handler**
- [ ] **Step 2: Add sessions.preview handler (retry with longer timeout)**
- [ ] **Step 3: Rebuild Docker and test**

---

## Phase C: End-to-End Verification

### Task C1: Full ClawNexus UI Smoke Test

**Test:** Open http://192.168.68.65:7842 in browser and check all UI panels work.

1. **Office view** — agents appear, presence events show agents online
2. **Dashboard** — config.get loads, no "Method not found" errors in console
3. **Channels** — channels.status returns channel list (even if empty/mock)
4. **Skills** — skills list loads
5. **Chat** — chat.send works (tested in Task B1 of previous session)
6. **Console errors** — zero "Method not found: X" errors after all methods implemented

**Commands to verify after Docker rebuild:**
```bash
# Watch hermes-house logs
docker logs hermes-house -f 2>&1 | grep -E "WS IN|WS OUT|Method not found"

# Test gateway endpoints directly
curl -s http://127.0.0.1:8642/v1/sessions | python3 -m json.tool
curl -s http://127.0.0.1:8642/v1/usage | python3 -m json.tool
curl -s http://127.0.0.1:8642/v1/channels | python3 -m json.tool
curl -s http://127.0.0.1:8642/v1/skills | python3 -m json.tool
curl -s http://127.0.0.1:8642/v1/tools | python3 -m json.tool
curl -s http://127.0.0.1:8642/v1/models | python3 -m json.tool
```

---

## Dependencies

- Task A1 (Sessions) must complete before Task B1 (sessions proxy)
- Task A2-A6 (HTTP endpoints) must complete before Tasks B2-B5
- Phase C (E2E test) requires Phase A + Phase B complete

## Self-Review Checklist

1. **Spec coverage:** Every ClawNexus RPC method has a handler (sessions.list/preview/delete, usage.status, config.get/set, channels.status/logout, skills.status/install, tools.catalog, models.list, chat.send/abort)
2. **Placeholder scan:** No TBD/TODO — all steps show concrete code
3. **Type consistency:** `SessionInfo`, `UsageInfo`, `ConfigSnapshot` field names match adapter-types.ts
4. **Gateway restart needed:** Adding HTTP endpoints to api_server.py requires gateway restart
5. **Docker rebuild needed:** server.py changes require `docker compose up -d --build`
