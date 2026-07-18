# Split Timeline (Forensics)

## Challenge Description
An investigation of a suspected data staging event on a finance workstation (`WS-0419`). We are provided with three raw files:
- `setupapi.dev.log`
- `usnjrnl.bin` (raw `$UsnJrnl:$J` extract)
- `mft.bin` (raw `$MFT` volume extract)

Our task is to reconstruct the staged data and extract the flag.

## Analysis & Solution
1. **Removable Media Identification**:
   Inspecting `setupapi.dev.log` reveals three USB storage devices connected to the machine during the session:
   - **SanDisk Cruzer Blade** (connected at `06:41:02`): Serial number `4C530001180529117094`
   - **SanDisk Ultra Fit** (connected at `07:13:11`): Serial number `AA010129180916122757`
   - **Kingston DataTraveler** (connected at `07:46:12`): Serial number `E0D55EA5730CF0507A2C0E1B`

2. **Parsing USN Journal**:
   By parsing `usnjrnl.bin` (which tracks NTFS file creations, deletions, and updates), we observe two distinct temporary file sessions in `$env:TEMP` containing `~df*.tmp` files:
   - **Session 1** (Starts `06:41:09`, right after the Cruzer Blade was inserted):
     Creates 8 temporary files: `~dfA31C.tmp`, `~df77E2.tmp`, `~df1B04.tmp`, `~df9C55.tmp`, `~df6D0B.tmp`, `~dfC418.tmp`, `~df3F9A.tmp`, `~df90E7.tmp`.
   - **Session 2** (Starts `07:13:12`, right after the Ultra Fit was inserted):
     Creates 6 temporary files: `~df2E81.tmp`, `~df4A19.tmp`, `~dfB730.tmp`, `~df08CD.tmp`, `~df5F62.tmp`, `~dfE394.tmp`.

3. **MFT Analysis & Staged Script Recovery**:
   Inspecting `mft.bin` reveals that a script named `stage.ps1` (Record 20) was created and subsequently deleted. Its resident `$DATA` attribute is still intact and contains the following logic:
   ```powershell
   # stage.ps1 -- block out $Target and park it in ADS until the courier run
   $s = (gwmi Win32_DiskDrive | ? {$_.InterfaceType -eq 'USB'} | select -f 1).SerialNumber
   # encrypt once over the whole file, then cut the ciphertext into blocks
   $c = RC4 $s ([IO.File]::ReadAllBytes($Target))
   for ($i = 0; $i -lt $c.Length; $i += 5) {
     $b = $c[$i..([Math]::Min($i + 4, $c.Length - 1))]
     $t = "$env:TEMP\~df{0:X4}.tmp" -f (Get-Random -Max 0xFFFF)
     sc $t 'tmp scratch block; superseded'
     [IO.File]::WriteAllBytes("${t}:sync", $b)
     $f = gi $t -Force
     $f.CreationTime = $Blend
     $f.LastWriteTime = $Blend
   }
   ```
   This script encrypts the target payload using RC4 with the inserted USB device's Serial Number as the key, splits it into 5-byte blocks, and stores each block in the `:sync` alternate data stream (ADS) of the temporary `~df*.tmp` files.

4. **Reconstructing and Decrypting the Payloads**:
   We locate the resident `:sync` streams (the second `$DATA` attributes of the corresponding MFT records) and concatenate the 5-byte chunks in their creation order:
   - **Session 1**:
     - Ciphertext (hex): `3d02f4f40a738c3400c53b9703e9c31458f1a19afea40f6d246bca5e56b1ccd98719fdbac460fab4`
     - Decryption Key (ASCII): `4C530001180529117094`
     - Decrypted plaintext: `staging selftest ok; no payload attached`
   - **Session 2**:
     - Ciphertext (hex): `a70e4a1f41d60334687cf396e70bf96577babc2877a6a83940c778f44a21`
     - Decryption Key (ASCII): `AA010129180916122757`
     - Decrypted plaintext: `athena{mft_records_tell_tales}`

## Flag
`athena{mft_records_tell_tales}`
