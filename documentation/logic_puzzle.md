# Logic Puzzle (Reverse Engineering)

## Challenge Description
An ELF 64-bit stripped executable `logic_puzzle.bin` that decrypts and outputs a flag when run with the correct input constraint.

## Disassembly Analysis
- **Argument Check**: The binary checks if `argc == 2`. If not, it calls `exit(1)`.
- **Constraint Check**:
  It parses `argv[1]` as an integer $X$ via `strtol()` and evaluates:
  $$(X - 1) \times X = 0\text{x}1b4178 \quad (1,786,232 \text{ in decimal})$$
  Solving this quadratic equation yields $X = 1337$.
- **Decryption Loop**:
  Using $X = 1337$ as a seed, the binary runs a 64-bit Linear Congruential Generator (LCG) loop 25 times to generate a key byte stream which is then XOR'd with the encrypted ciphertext bytes stored at offset `0x2010`:
  - $X = (X \times 0\text{x}5851f42d4c957f2d + 0\text{x}14057b7ef767814f) \pmod{2^{64}}$
  - $key = (X \gg 33) \pmod{256}$
  - $plaintext\_byte = ciphertext\_byte \oplus key$

## Verification & Flag
Running the binary with candidate input `1337` prints the flag:
```bash
$ ./logic_puzzle.bin 1337
athena{logic_puzz1e_4c7b}
```

Flag: `athena{logic_puzz1e_4c7b}`
