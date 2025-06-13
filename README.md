
# THM-NetworkingSecureProtocols

**TryHackMe - Networking Secure Protocols Lab**
Secure networking protocols (TLS, HTTPS, SSH, VPN) analysis using Wireshark, OpenSSL, Chromium `--ssl-key-log-file`, `ssh`, `sftp`, `openvpn`, and `wg-quick`, showcasing secure session inspection, packet decryption, and encrypted tunneling.

**By Ramyar Daneshgar**

---

## Task 1: TLS – Transport Layer Security

I began by reviewing how TLS secures client-server communications. TLS operates at the Transport Layer and was designed to replace SSL due to its cryptographic weaknesses. TLS 1.3 removes insecure algorithms like RSA key exchange and static Diffie-Hellman, reducing attack surface and enabling forward secrecy by default.

To confirm the protocol TLS is based on, I selected **SSL**.

To analyze TLS behavior:

* I launched **Wireshark** and captured traffic on TCP port 443.
* I used **OpenSSL** to initiate a test connection and inspect the server’s certificate chain:

  ```bash
  openssl s_client -connect tryhackme.com:443
  ```

This allowed me to verify the server presented a valid X.509 certificate signed by a trusted CA. I noted that self-signed certificates were not verifiable via a public trust chain.

**Answer:** SSL
**Answer:** self-signed certificate

---

## Task 2: HTTPS – HTTP Over TLS

Next, I analyzed HTTPS (HTTP secured by TLS). I first inspected a normal HTTP request and observed how it transmits in plaintext over port 80. Then, I captured HTTPS traffic over port 443 and walked through the handshake.

**Steps Taken:**

1. Captured traffic using Wireshark while browsing an HTTPS page.
2. Observed the three-step TCP handshake (SYN, SYN-ACK, ACK).
3. Counted 8 packets used in TLS negotiation (ClientHello, ServerHello, certificate exchange, key exchange, and Finished).
4. Verified that the actual HTTP request (e.g., `GET /login`) was encrypted and encapsulated as TLS application data.

To decrypt the HTTPS session:

* I launched Chromium with session key logging enabled:

  ```bash
  chromium --ssl-key-log-file=~/ssl-key.log
  ```
* In Wireshark:

  * Navigated to *Preferences > Protocols > TLS*
  * Set the pre-master secret log file path
* Reloaded the `.pcapng` file and decrypted the stream.

I located the `GET /login` request in **packet 10** and verified the total **TLS handshake took 8 packets**.

After decryption, I inspected **packet 366**, where plaintext credentials were recovered.

**Answer:** 8
**Answer:** 10
**Answer:** THM{B8WM6P}

---

## Task 3: Securing Email Protocols – SMTPS, POP3S, IMAPS

To examine secure email communications, I compared plaintext and TLS-encrypted variants of SMTP, POP3, and IMAP.

| Protocol | Plain Port | TLS Port |
| -------- | ---------- | -------- |
| SMTP     | 25         | 465/587  |
| POP3     | 110        | 995      |
| IMAP     | 143        | 993      |

**Steps Taken:**

* Captured IMAP traffic on port 143 using Wireshark.
* Applied the following filter to locate plaintext credentials:

  ```bash
  tcp.port == 143 && frame contains "LOGIN"
  ```

I observed base64-encoded login attempts, proving that credentials could be intercepted if TLS is not enforced.

**Answer:** IMAP

---

## Task 4: SSH – Secure Shell

To assess SSH, I connected to a remote host using:

```bash
ssh user@targethost
```

SSH encrypts both the authentication and data stream using ciphers negotiated in the handshake. It replaces Telnet (port 23), which transmits credentials in plaintext.

**Advanced Features Tested:**

* X11 forwarding:

  ```bash
  ssh -X user@host
  ```
* Port forwarding (used to tunnel internal services securely):

  ```bash
  ssh -L 8080:localhost:80 user@host
  ```

These tests confirmed SSH’s flexibility for secure remote administration, tunneling, and GUI access over encrypted sessions.

**Answer:** OpenSSH

---

## Task 5: SFTP and FTPS – Secure File Transfers

I compared SFTP and FTPS:

| Protocol | Transport | Port |
| -------- | --------- | ---- |
| SFTP     | SSH       | 22   |
| FTPS     | TLS       | 990  |

**Steps Taken:**

* Initiated a secure file transfer via:

  ```bash
  sftp user@host
  ```

SFTP inherits its encryption from SSH and does not require separate control/data channels or certificate validation. FTPS, by contrast, is more complex due to TLS negotiation, certificate handling, and firewall configurations.

---

## Task 6: VPN – Virtual Private Network

To simulate secure remote access and IP masking, I configured and tested both OpenVPN and WireGuard.

**OpenVPN:**

```bash
sudo openvpn --config client.ovpn
```

**WireGuard:**

```bash
sudo wg-quick up wg0
```

**Verification Steps:**

1. Checked new IP address with:

   ```bash
   dig +short myip.opendns.com @resolver1.opendns.com
   ```
2. Ensured DNS requests were routed over the VPN and not leaking via local resolvers.
3. Used `dnsleaktest.com` to validate no DNS leak.

**Answer:** VPN

I confirmed that VPN tunnels encrypted all traffic and masked the client IP, emulating access from the VPN server’s location. I noted that split tunneling or misconfigured DNS could lead to leakage.

---

## Task 7: Final Challenge

After following the provided instructions on the external site, I obtained the flag.

**Answer:** THM{Protocols\_secur3d}

---

## Final Thoughts and Lessons Learned

1. **TLS is a backward-compatible layer that can secure plaintext protocols without modifying their structure**, enabling easy integration with HTTP, IMAP, SMTP, and others.

2. **Certificate validation is foundational** to authentication. Trusting self-signed certificates undermines security entirely.

3. **Wireshark with session key logging** provides a powerful method for analyzing encrypted traffic in a lab setting. This is invaluable for protocol forensics, incident response, and testing.

4. **SSH is a multipurpose protocol** supporting secure logins, file transfers, port forwarding, and GUI tunneling — all encrypted end-to-end.

5. **SFTP simplifies secure file transfers** compared to FTPS, which requires complex TLS setup and certificate handling.

6. **VPNs encrypt traffic and anonymize users**, but require validation to ensure they’re routing and securing traffic as expected. Misconfigurations such as split tunneling or DNS leaks reduce their effectiveness.

7. **Transport-layer cryptography is not enough** if endpoint security is compromised. TLS key logs highlight that client-side control of encryption is a critical attack surface.

