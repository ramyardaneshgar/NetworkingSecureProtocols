# THM-NetworkingSecureProtocols
TryHackMe- Networking Secure Protocols Lab 

Secure networking protocols (TLS, HTTPS, SSH, VPN) analysis using Wireshark, OpenSSL, Chromium --ssl-key-log-file, ssh, sftp, openvpn, and wg-quick, showcasing secure session inspection, packet decryption, and encrypted tunneling.

By Ramyar Daneshgar 


---

## Task 1: TLS – Transport Layer Security

TLS is a protocol operating at the Transport Layer that encrypts communications between clients and servers. It ensures confidentiality, integrity, and authenticity of data in transit. TLS is the successor to SSL and has deprecated older insecure cryptographic primitives, with TLS 1.3 removing support for RSA key exchange and static DH.

**Answer:** SSL

### Tools and Methods:

* **Wireshark** to analyze TLS handshake, cipher negotiation, and encrypted application data over TCP.
* **OpenSSL** to inspect server certificates and test cipher support:

  ```bash
  openssl s_client -connect targetdomain:443
  ```

TLS relies on X.509 certificates issued by Certificate Authorities (CAs). Self-signed certificates break the chain of trust and are not accepted by modern clients in production.

**Answer:** self-signed certificate

---

## Task 2: HTTPS – HTTP Over TLS

HTTPS is implemented by layering TLS above HTTP. After a standard TCP three-way handshake, a TLS handshake is initiated. Only once this session is negotiated are encrypted HTTP requests (such as GET or POST) transmitted.

### Wireshark Breakdown:

1. TCP Handshake: SYN, SYN-ACK, ACK (3 packets)
2. TLS Handshake: Certificate exchange, key agreement, cipher suite negotiation (8 packets total)
3. Encrypted HTTP (Application Data) over port 443

**Answer:** 8
**Answer:** 10

To inspect encrypted application traffic, I enabled key logging and imported the secrets into Wireshark.

### Decryption Procedure:

1. Launch Chromium with:

   ```bash
   chromium --ssl-key-log-file=~/ssl-key.log
   ```
2. In Wireshark:

   * Preferences > Protocols > TLS
   * Set Pre-Master-Secret log file to `~/ssl-key.log`
3. Reload the packet capture; HTTP payloads decrypted

Packet 366 in the decrypted capture revealed cleartext credentials.

**Answer:** THM{B8WM6P}

---

## Task 3: Securing Email Protocols – SMTPS, POP3S, IMAPS

Legacy mail protocols transmit credentials and messages in plaintext. TLS is applied to secure communication, creating SMTPS, POP3S, and IMAPS.

| Protocol | Plain Port | Secure Port |
| -------- | ---------- | ----------- |
| SMTP     | 25         | 465, 587    |
| POP3     | 110        | 995         |
| IMAP     | 143        | 993         |

Unencrypted IMAP transmits authentication commands and passwords over the wire.

**Answer:** IMAP

Using Wireshark:

```bash
tcp.port == 143 && frame contains "LOGIN"
```

This confirms the presence of unencrypted login credentials on standard IMAP traffic.

---

## Task 4: SSH – Secure Shell

SSH provides encrypted shell access and file transfer over untrusted networks. It replaces Telnet, which lacks encryption and exposes credentials and session data.

**Answer:** OpenSSH

SSH authentication mechanisms:

* Password-based
* Public key (`~/.ssh/id_ed25519`)
* Agent forwarding and MFA extensions

### Commands Used:

```bash
ssh user@targethost
```

For X11 forwarding (GUI apps):

```bash
ssh -X user@targethost
```

SSH can also tunnel TCP traffic:

```bash
ssh -L 8080:localhost:80 user@targethost
```

This redirects local port 8080 to remote port 80, useful for internal web access.

---

## Task 5: SFTP and FTPS – Secure File Transfers

SFTP is a file transfer subsystem of SSH. It is not FTP over SSH. FTPS is FTP wrapped in TLS.

| Protocol | Secure Transport | Port |
| -------- | ---------------- | ---- |
| SFTP     | SSH              | 22   |
| FTPS     | TLS              | 990  |

### Command Used:

```bash
sftp user@host
```

This initiates an SSH-secured file transfer session. Unlike FTPS, SFTP does not require negotiating separate control and data connections.

---

## Task 6: VPN – Virtual Private Network

VPNs create encrypted tunnels between clients and remote networks. VPNs encrypt the entire IP packet, allowing for secure remote access and obfuscation of the source IP.

Common VPN protocols:

* IPsec (AH, ESP modes)
* OpenVPN (TLS over UDP)
* WireGuard (Curve25519-based)

**Answer:** VPN

### Commands Used:

* OpenVPN:

  ```bash
  sudo openvpn --config client.ovpn
  ```
* WireGuard:

  ```bash
  sudo wg-quick up wg0
  ```

### Verification:

To verify VPN routing and confirm IP address spoofing:

```bash
dig +short myip.opendns.com @resolver1.opendns.com
```

This reveals the public IP assigned by the VPN server.

VPN misconfigurations such as split tunneling or DNS leaks can reveal the true IP address or route traffic outside the VPN tunnel. Use tools like `dnsleaktest.com` to verify.

---

## Task 7: Final Challenge

After accessing the provided test environment, I submitted the correct response.

**Answer:** THM{Protocols\_secur3d}

---

## Final Thoughts and Lessons Learned

1. **Transport Layer Security is extensible and backwards-compatible**, enabling encrypted communications without altering application or network-layer protocols.

2. **Certificate trust chains are critical.** Misuse of self-signed certificates in production creates a false sense of security and invites impersonation attacks.

3. **Wireshark and OpenSSL** are essential tools for auditing, verifying, and decrypting TLS-secured traffic in controlled environments.

4. **Port-level protocol fingerprinting is essential for secure network architecture.** Blocking port 23, 21, 143, and 110 in favor of their secure counterparts is baseline practice.

5. **SSH is a versatile protocol** that supports remote login, file transfer, port forwarding, and graphical application access—all over an encrypted channel.

6. **VPNs must be validated post-connection** to ensure full-tunnel routing, no DNS leakage, and IP masking are in effect.

7. **TLS session secrets must be tightly controlled.** Once exposed (e.g., via `--ssl-key-log-file`), full session contents can be decrypted. Endpoint security remains a vital enforcement point.
