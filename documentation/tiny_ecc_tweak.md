# Tiny ECC Tweak (Crypto)

## Challenge Description
A cryptographic challenge involving ECDSA signatures on the secp256k1 curve. A whistleblower exfiltrated a log file containing two authenticated transmissions signed by the Syndicate's central command node, along with an encrypted payload. The signing system suffered from a broken random number generator, leading to nonce reuse in the signatures.

## Vulnerability & Analysis
1. **ECDSA Signature Reuse**:
   The log file `pub.txt` contains two ECDSA signatures with identical `r` values:
   - `sig1_r = 85648066978054117297931732228825879776919476959248536209880673617777258551576`
   - `sig2_r = 85648066978054117297931732228825879776919476959248536209880673617777258551576`
   
   This indicates that the same nonce `k` was used for both signatures, which is a critical vulnerability in ECDSA.

2. **ECDSA Signature Equation**:
   For ECDSA, the signature (r, s) is computed as:
   - `r = (k * G).x mod n`
   - `s = k^(-1) * (h + r * d) mod n`
   
   Where:
   - `k` is the random nonce
   - `h` is the hash of the message
   - `d` is the private key
   - `n` is the curve order

3. **Nonce Recovery**:
   Given two signatures with the same `r` (and thus same `k`):
   - `s1 = k^(-1) * (h1 + r * d) mod n`
   - `s2 = k^(-1) * (h2 + r * d) mod n`
   
   Solving for `k`:
   - `k = (h1 - h2) / (s1 - s2) mod n`

## Exploitation
1. **Message Hashing**:
   Compute SHA-256 hashes of the two messages:
   - `msg1 = 'public message alpha'` → `h1`
   - `msg2 = 'public message beta'` → `h2`

2. **Recover Nonce k**:
   Using the formula `k = (h1 - h2) * (s1 - s2)^(-1) mod n`
   - Recovered `k = 58297298662004539087198126476885464609896899561172077532306668666516937268200`

3. **Recover Private Key d**:
   Using the formula `d = (s1 * k - h1) * r^(-1) mod n`
   - Recovered `d = 27448684403655542201535540966842138522806071396365634423407980633521445332433`
   - Hex: `0x3caf67a22ee440123acb9e50d3c7bb63ba21afeef0eb3ba0ed81796878f2e5d1`

4. **Verification**:
   The recovered private key was verified by computing the corresponding public key and matching it against the provided public key coordinates:
   - `pubkey_x = 0x702468aa105b856bc210955eb43be9ce4a04f4c5e4ea5c2571515e2d3add7223`
   - `pubkey_y = 0xbf26618e324947c764ea50b36f3a51a6a0149be0a27ee23942ce7eeb176d09ba`

5. **AES Decryption**:
   The encrypted payload uses AES-GCM with:
   - Key derived as SHA-256 of the private key
   - Nonce: `0x429f5788ac9f95a7ff00b71b`
   - Ciphertext: `0x8af0c838f666ff56951dc7a72cc4ebd3cbcc7a08564d9d4a6268ecbc2cd6233ae460fd8fe91a35`
   
   The last 16 bytes of the ciphertext are the authentication tag. After decryption, the plaintext reveals the flag.

## Flag
`athena{n0nc3_r3us3_3cc}`
