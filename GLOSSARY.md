# Glossary

- **BMC (baseboard management controller):** independent SoC on a server motherboard providing out-of-band management.
- **Out-of-band management:** managing a system through a channel independent of its main OS and network stack.
- **IPMI:** Intelligent Platform Management Interface; legacy management protocol, version 2.0 widely deployed.
- **RMCP+:** the IPMI 2.0 LAN transport over UDP port 623.
- **RAKP:** Remote Authenticated Key-Exchange Protocol used by IPMI 2.0; discloses an HMAC of the password to unauthenticated requesters.
- **Cipher suite 0:** IPMI suite with no authentication, integrity or encryption; allowed login without a valid password on affected BMCs.
- **Redfish:** DMTF RESTful management API over HTTPS with JSON/OData resources.
- **Redfish Host Interface:** in-band path from the host OS to the BMC's Redfish service over a virtual NIC.
- **KCS:** Keyboard Controller Style interface used by the host OS to send IPMI commands to the BMC.
- **Shared LOM:** configuration where BMC traffic shares a host network port via a sideband.
- **Virtual media:** BMC feature that presents a remote image as a USB drive to the host.
- **SEL:** IPMI System Event Log.
- **iDRAC / iLO / XClarity Controller:** Dell, HPE and Lenovo BMC product lines.
- **OpenBMC:** Linux Foundation open-source BMC firmware stack.
- **AHB bridge:** internal bus bridge in ASPEED BMCs; misconfigured bridges enabled the Pantsdown host-to-BMC attack.
- **SPDM:** DMTF Security Protocol and Data Model for device authentication and measurement.
- **Hardware root of trust:** dedicated component that verifies firmware before it runs.
- **KEV:** CISA's Known Exploited Vulnerabilities catalogue.
