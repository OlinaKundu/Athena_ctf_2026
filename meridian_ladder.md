# Meridian Ladder (Web)

## Challenge Description
A Node.js/Express financial web application managing fixed-income ladders. The objective is to unwind the protected "House Reserve Ladder" (`eyJ0IjoibGFkZGVyIiwibiI6MX0`), which is normally locked and ineligible for early unwind.

## Vulnerability & Analysis
1. **API Endpoints**:
   Examining `/static/app.js` reveals several endpoints, including:
   - `PATCH /api/preferences` (Updates settings/preferences)
   - `GET /api/ladder/<id>` (Retrieves details and metadata for a ladder)
   - `POST /api/ladder/<id>/unwind` (Unwinds a ladder early)
2. **Prototype Pollution**:
   The `PATCH /api/preferences` endpoint parses JSON body and recursively merges it into the user settings object. Because it does not sanitize key names, it is vulnerable to **Prototype Pollution**.
3. **Logic Bypass**:
   When fetching ladder metadata, the backend checks the policy configuration:
   `policy.allowEarlyUnwind`
   The "House Reserve Ladder" has a policy tier of `reserve` and does not define the `allowEarlyUnwind` property at all. 
   By sending a prototype pollution payload:
   ```json
   {
     "__proto__": {
       "allowEarlyUnwind": true
     }
   }
   ```
   We pollute `Object.prototype.allowEarlyUnwind = true`. Consequently, the check on the House Reserve Ladder resolves to `true` via the prototype chain lookup, making it eligible for early unwind (`unwindEligible: true`).

## Exploitation
Sending a `POST` request to `/api/ladder/eyJ0IjoibGFkZGVyIiwibiI6MX0/unwind` after the prototype pollution succeeds and returns the flag in the settlement token field.

## Flag
`athena{dN06Ub8zKq4hxaUa}`
