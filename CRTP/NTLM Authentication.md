---
title: NTLM Authentication
service: TCP
---
# ¿Qué es?

Aside from Kerberos and LDAP, Active Directory uses several other authentication methods which can be used (and abused) by applications and services in AD. These include LM, NTLM, NTLMv1, and NTLMv2. LM and NTLM here are the hash names, and NTLMv1 and NTLMv2 are authentication protocols that utilize the LM or NT hash. Below is a quick comparison between these hashes and protocols, which shows us that, while not perfect by any means, Kerberos is often the authentication protocol of choice wherever possible. It is essential to understand the difference between the hash types and the protocols that use them.

|**Hash/Protocol**|**Cryptographic technique**|**Mutual Authentication**|**Message Type**|**Trusted Third Party**|
|---|---|---|---|---|
|`NTLM`|Symmetric key cryptography|No|Random number|Domain Controller|
|`NTLMv1`|Symmetric key cryptography|No|MD4 hash, random number|Domain Controller|
|`NTLMv2`|Symmetric key cryptography|No|MD4 hash, random number|Domain Controller|
|`Kerberos`|Symmetric key cryptography & asymmetric cryptography|Yes|Encrypted ticket using DES, MD5|Domain Controller/Key Distribution Center (KDC)|

## LM

LAN Manager (LM) hashes are an outdated and insecure password storage mechanism used in older Windows operating systems, first introduced in 1987 on OS/2. Stored in the SAM database or NTDS.DIT on Domain Controllers, LM hashes have been disabled by default since Windows Vista/Server 2008 due to weaknesses in their algorithm but may still be found in legacy systems.

LM hashes limit passwords to 14 uppercase characters, reducing the keyspace and making them easy to crack with tools like [[Hashcat]]. Passwords are split into two 7-character chunks, hashed separately using DES and concatenated, allowing attackers to brute-force each half independently. Group Policy can disable the use of LM hashes, which were previously stored alongside NTLM hashes in systems prior to Windows Vista/Server 2008.

## NTLM

`NT LAN Manager` (NTLM) hashes are used on modern Windows systems. It is a challenge-response authentication protocol and uses three messages to authenticate: a client first sends a `NEGOTIATE_MESSAGE` to the server, whose response is a `CHALLENGE_MESSAGE` to verify the client's identity. Lastly, the client responds with an `AUTHENTICATE_MESSAGE`. These hashes are stored locally in the SAM database or the NTDS.DIT database file on a Domain Controller. The protocol has two hashed password values to choose from to perform authentication: the LM hash (as discussed above) and the NT hash, which is the MD4 hash of the little-endian UTF-16 value of the password. The algorithm can be visualized as: `MD4(UTF-16-LE(password))`.

NTLM hashes are significantly stronger than LM hashes, supporting the full Unicode character set (65,536 characters), but they remain vulnerable to offline brute-force attacks using tools like Hashcat. Modern GPU attacks can brute-force the entire 8-character NTLM keyspace in under 3 hours, and longer passwords can also be cracked using dictionary attacks combined with rules. NTLM hashes are particularly susceptible to pass-the-hash attacks, allowing attackers to use the hash to authenticate without knowing the password. An NT hash represents the second half of the NTLM hash and looks like this: `b4b9b02e6f09a9bd760f388b67351e2b`, while a full NTLM hash includes additional user-specific data, such as: `Rachel:500:aad3c435b514a4eeaad3b935b51304fe:e46b9e548fa0d122de7f59fb6d48eaa2:::`.

```bash
crackmapexec smb 10.129.41.19 -u rachel -H e46b9e548fa0d122de7f59fb6d48eaa2
```

## NTLMv1 (Net-NTLMv1)

NTLMv1 is an outdated authentication protocol using a challenge/response mechanism based on NT and LM hashes. The server sends an 8-byte challenge, and the client responds with a 24-byte value derived using DES encryption. While it cannot be used for pass-the-hash attacks, NTLMv1 is vulnerable to offline cracking and relay attacks. Its weaknesses led to the development of the more secure NTLMv2.

## NTLMv2 (Net-NTLMv2)

NTLMv2, introduced in Windows NT 4.0 SP4 and default since Windows Server 2000, is a more secure alternative to NTLMv1. It protects against spoofing attacks by using a robust challenge/response mechanism involving HMAC-MD5. The protocol sends two responses to an 8-byte server challenge: one based on a client-generated random challenge and another containing a variable-length client challenge with a timestamp, random value, and domain name. This strengthens security by incorporating additional data into the hashing process, making NTLMv2 significantly harder to crack. An NTLMv2 hash example looks like: `admin::N46iSNekpT:08ca45b7d7ea58ee:88dcbe4446168966a153a0064958dac6:5c7830315c7830310000000000000b45c67103d07d7b95acd12ffa11230e0000000052920b85f78d013c31cdb3b92f5d765c783030`.

## Domain Cached Credentials (MSCache2)

Domain Cached Credentials (DCC), or MS Cache, allow domain-joined hosts to authenticate users locally if a Domain Controller is unreachable. Stored in the registry, these hashes are slow to crack and cannot be used for pass-the-hash attacks. They retain the last ten domain user hashes, e.g., `$DCC2$10240#bjones#e4e938d12fe5974dc42a90120bd9c90f`. Understanding hash types and their limitations is crucial for effective penetration testing in AD environments.