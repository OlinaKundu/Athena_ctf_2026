# Mailroom Echo (DF/Forensics)

## Challenge Description
An email file `mailroom_echo.eml` exported from quarantine, which contains a MIME attachment holding data routed secretly out of the building.

## Analysis & Solution
1. **Email Structure**:
   Analyzing `mailroom_echo.eml` reveals a standard MIME email format with a Base64-encoded attachment named `quarterly_summary.txt`:
   ```text
   UXVhcnRlcmx5IHJlY29uY2lsaWF0aW9uIGNvbXBsZXRlLgpCYWNrdXAgbWFya2VyOiBhdGhlbmF7bWltZV90aHJlYWRzX3JldmVhbF90aGVfdHJ1dGh9CkRvIG5vdCBmb3J3YXJkLgo=
   ```
2. **Decoding**:
   Decoding the Base64 payload reveals the plain text attachment content:
   ```text
   Quarterly reconciliation complete.
   Backup marker: athena{mime_threads_reveal_the_truth}
   Do not forward.
   ```

## Flag
`athena{mime_threads_reveal_the_truth}`
