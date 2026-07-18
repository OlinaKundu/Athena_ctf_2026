# Upload Lantern

This challenge was about finding a flag through a path traversal vulnerability.

## Solution

1. We checked the source code. The active sanitizer was `sanitize_legacy`.
2. The legacy sanitizer only replaces forward slashes `/`. It does not replace backward slashes `\`.
3. The server runs on Windows, so Python treats backward slashes `\` as path separators.
4. The flag is located at `/srv/app/private/private_flag.txt`.
5. We used a path traversal payload with backward slashes: `..\private\private_flag.txt`.
6. We sent a GET request to `/view?name=..\private\private_flag.txt`.
7. This bypassed the sanitizer and returned the flag.

## Flag

athena{bT33103rgBZtm8n9}
