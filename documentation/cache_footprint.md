# Cache Footprint

This challenge was about finding a flag from a deleted database entry.

## Solution

1. We opened the database file `browser_history.sqlite`.
2. We looked at the tables. The table `downloads` was missing ID number 6.
3. We checked the raw file. We found deleted text for ID number 6.
4. The deleted text had a file path `/tmp/notes.txt`.
5. The text had a base64 string: `Ch0HFgVMS0dAVQIdCiwASEBAbk0DDDAQB1hVSQ==`.
6. We found the key `kiosk-0419` inside a cookie.
7. We decoded the base64 string.
8. We decrypted the bytes using XOR with the key `kiosk-0419`.
9. The result was the flag.

## Flag

athena{sqlite_kept_the_clue}
