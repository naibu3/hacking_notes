---
title: "{title}"
service: TCP
---
# ¿Qué es?

Active Directory supports [Lightweight Directory Access Protocol (LDAP)](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol) for directory lookups. LDAP is an open-source and cross-platform protocol used for authentication against various directory services (such as AD). The latest LDAP specification is [Version 3](https://tools.ietf.org/html/rfc4511), published as RFC 4511. A firm understanding of how LDAP works in an AD environment is crucial for attackers and defenders. LDAP uses port 389, and LDAP over SSL (**LDAPS**) communicates over port **636**.

# Configuración

# Auditando el servicio

## ldapsearch

```bash
ldapsearch -x -H ldap://10.10.11.35 -b "DC=cicada,DC=htb"
```

