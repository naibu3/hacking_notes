---
title: "{title}"
service: TCP
---
# ¿Qué es?

Kerberos is a widely used authentication protocol in Active Directory environments due to its security, efficiency, and ability to support mutual authentication. Operates on **port 88 (TCP/UDP)**.

nstead of transmitting passwords over the network, it uses encrypted tickets to authenticate users and grant access to resources. When a user logs in, their password is hashed and used to request a Ticket Granting Ticket (TGT) from the Key Distribution Center (KDC). The TGT serves as proof of identity and allows the user to request a Service Ticket (TGS) for specific resources. The TGS is then presented to the desired service, which validates it and grants access if valid. Kerberos operates over port 88 (TCP/UDP), supports interoperability with other systems using the protocol, and is stateless, relying on tickets rather than session tracking. This ensures secure, efficient authentication in networked environments, particularly in Active Directory domains.

# Configuración

# Auditando el servicio