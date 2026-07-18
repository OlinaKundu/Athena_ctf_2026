# Table Stakes

This challenge was about reversing an ELF binary program.

## Solution

1. We checked the binary file `challenge.bin`. It was a 64-bit ELF file.
2. The program checks if our input flag is correct.
3. The flag must be exactly 27 characters long.
4. The program uses a function to encode our flag.
5. The encoding function does two things to each character:
   - It rotates the bits of the character left by 3 positions.
   - It XORs the result with a byte from a dynamic key.
6. The key starts at value `0x5f`. It updates in a loop using a mathematical formula.
7. We found the correct encoded bytes inside the program's memory.
8. We wrote a script to reverse the steps.
   - It XORs the correct bytes with the dynamic key.
   - It rotates the bits right by 3 positions.
9. This gave us the original flag.

## Flag

athena{st4tic_str1ngs_b0rn}
