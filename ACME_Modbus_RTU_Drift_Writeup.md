# ACME Industrial -- Modbus RTU Drift Extraction Writeup

## Challenge Overview

This challenge provided a firmware image recovered from an ACME
Industrial Modbus RTU controller together with a short note identifying
the firmware version and string table offset. The objective was to
reverse the bootloader's configuration de-obfuscation routine and
recover the hidden drift parameter embedded inside the firmware.

## Initial Analysis

I began by inspecting the supplied firmware and accompanying notes. The
recovered metadata indicated:

-   Vendor: ACME Industrial
-   Firmware Version: 3.7.1
-   String Table Offset: 0x40

Rather than searching only for printable strings, I concentrated on
identifying the routine responsible for decoding the stored
configuration.

## Reverse Engineering

After locating the configuration handling logic, I reconstructed the
decoding routine.

The algorithm consisted of two simple operations:

1.  XOR each byte with the repeating key `KEY42`.
2.  Rotate the result right by three bits (ROR 3).

Equivalent pseudocode:

``` c
decoded = ror8(encoded ^ key[i % 5], 3);
```

## Recovering the Configuration

Applying the decoding routine to the configuration blob produced:

``` text
sys_version=3.7.1
vendor=ACME
drift_payload=athena{scad4_firmware_root}
```

The hidden drift parameter was therefore:

``` text
athena{scad4_firmware_root}
```

## Conclusion

This firmware did not use encryption for its configuration. Instead, it
relied on a reversible obfuscation routine based on a repeating XOR key
and a bit rotation. Reversing that logic allowed me to recover the
embedded configuration and extract the hidden drift parameter.

## Flag

``` text
athena{scad4_firmware_root}
```
