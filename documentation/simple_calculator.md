# Simple Calculator

This challenge was about reversing an Android application file (.apk).

## Solution

1. We checked the APK file `illogical.apk`.
2. We parsed the `classes.dex` file inside the APK.
3. We found a method named `revealFlag` inside the class `MainActivity`.
4. We disassembled the bytecode of this method.
5. The method does these steps:
   - It XORs two integer arrays `MainActivity.c` and `g9.a` to make an 8-byte key.
   - It hashes the key using SHA-256 to get an AES-128 key.
   - It uses `MainActivity.d` as the IV (initialization vector).
   - It decrypts the ciphertext array `MainActivity.e` using AES-128-CBC.
6. We dumped the static initializers of the class to get all these integer arrays.
7. We wrote a Python script to do the same decryption logic.
8. The decrypted text revealed the flag.

## Flag

athena{2850289b2ace6865dc26fccf571b1f2a}
