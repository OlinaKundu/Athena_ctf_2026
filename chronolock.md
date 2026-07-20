# Chronolock (Reverse Engineering)

## Challenge Description
A 64-bit ELF executable `chronolock` that derives a decryption key by running a loop for a huge number of iterations before decrypting and printing the flag. Running the binary directly would take over a century.

## Analysis & Key Derivation Bypass
- **The Long Loop**:
  - The binary contains a loop that initializes a counter in `rcx` to `0xf9ccd8a1c5080000` (about 18 quintillion) and decrements it by 8 in each iteration. This runs for exactly $2.25 \times 10^{18}$ iterations.
  - In each iteration, the binary adds the golden ratio constant `0x9e3779b97f4a7c15` eight times to the `rax` register (initially `0x00c0ffeed00d1337`).
- **Mathematical Bypass**:
  - Instead of running the loop, we can compute the final value of `rax` directly using multiplication modulo $2^{64}$:
    
    $$final_rax = (initial_rax + 0xf9ccd8a1c5080000 × 0x9e3779b97f4a7c15) mod 2⁶⁴$$
    
    $$final_rax = 0xc1c70cf3d9b51337$$
    
  - The binary hashes `final_rax` using the SplitMix64 algorithm and verifies that the output matches `0x7defb1ac745c0756`. Since the values match, we can bypass the loop entirely.

## Flag Reconstruction
- **Constant Keys**:
  - The decryption key used for the flag reconstruction is the byte `0x5a`, which is stored in the `.data` segment at offset `0x43b0`.
- **Decryption Logic**:
  - The binary constructs the flag by loading encrypted chunks from the `.rodata` segment, XORing them with `0x5a`, and placing the characters in specific stack offsets (`rbp-0x52` down to `rbp-0x29`).
  - We wrote a Python script to extract these raw bytes, apply the XOR and stack mapping logic, and print the flag.

## Verification & Flag
Running the Python solve script `solve.py` yields the reconstructed flag:
```bash
$ python solve.py
eax = 0x5a
Flag: athena{4_s1ngl3_byt3_fl1p_s4v3s_c3ntur13s}
```

Flag: `athena{4_s1ngl3_byt3_fl1p_s4v3s_c3ntur13s}`
