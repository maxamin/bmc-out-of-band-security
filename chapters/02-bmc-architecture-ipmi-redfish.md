# BMC Architecture, IPMI and Redfish

To secure a BMC you need to know what it runs, which interfaces it exposes and what each protocol trusts. This chapter describes a typical BMC from silicon to API and compares the two main management protocols, IPMI and Redfish.

## Hardware and firmware

A typical x86 server BMC is an ARM SoC such as the ASPEED AST2500 or AST2600 with its own DRAM, SPI flash holding the firmware, a dedicated Ethernet MAC (often also wired to a sideband on the host NIC for "shared LOM" operation), a video controller that provides the host's VGA output for remote KVM, and connections to the host:

| Host link | Used for | Security note |
|---|---|---|
| LPC / eSPI | KCS IPMI interface, POST codes, sometimes SPI flash access | Host-side access without network credentials |
| PCIe | VGA, BMC-host bridges | Public research (Pantsdown, 2019) showed bridges allowing host read/write of BMC memory |
| USB | Virtual media, virtual keyboard/mouse, Redfish Host Interface NIC | Host can reach BMC APIs locally |
| I2C / SMBus | Sensors, power supplies, PCIe devices, FRU EEPROMs | Wide reach across the board |
| SPI | Host BIOS flash (often muxed) | BMC can reflash host firmware |

Firmware is usually embedded Linux (OpenBMC, or a vendor distribution) running a web server, IPMI daemon, Redfish service, SSH, KVM and virtual-media services, SNMP agent and update service. Vendor products include Dell iDRAC, HPE iLO, Lenovo XClarity Controller and Supermicro BMC firmware, much of the latter historically built on AMI MegaRAC. OpenBMC, a Linux Foundation project, is used by hyperscalers and increasingly by vendors.

Matrosov, Rodionov and Bratus note in *Rootkits and Bootkits* that these embedded web servers are typically written in C, including CGI handlers, and are therefore prime targets for input-handling bugs.

## IPMI

The Intelligent Platform Management Interface was defined by Intel, HP, NEC and Dell in 1998; version 2.0 (2004) is what most servers implement. Over the network it runs as RMCP+ on UDP port 623. Locally the OS reaches it through KCS or similar interfaces, typically with `ipmitool` or `ipmiutil`.

IPMI 2.0 authentication uses the RAKP (Remote Authenticated Key-Exchange Protocol) handshake. Sessions select a *cipher suite* that combines authentication, integrity and confidentiality algorithms. Suite 3 (HMAC-SHA1, HMAC-SHA1-96, AES-CBC-128) and suite 17 (HMAC-SHA256 variants) are common secure choices; suite 0 means no authentication, no integrity and no encryption. Chapter 3 explains why both RAKP and suite 0 became serious problems.

Common functions exposed: chassis power control, sensor data records, system event log (SEL), serial-over-LAN, user management, LAN configuration and FRU data. IPMI was designed for trusted management networks and shows its age: limited password lengths (16 or 20 bytes depending on version), weak role model, and no modern transport security.

## Redfish

Redfish is the DMTF standard intended to replace IPMI, first published in 2015. It is a RESTful API over HTTPS with JSON payloads following OData conventions, rooted at `/redfish/v1/`. Resources model systems, chassis, managers (the BMC itself), accounts, sessions, event subscriptions, firmware inventory and update services.

```bash
# Read-only queries against your own BMC
curl -sk -u admin:'<pw>' https://bmc.example.lab/redfish/v1/ | jq .
curl -sk -u admin:'<pw>' https://bmc.example.lab/redfish/v1/Managers | jq .
curl -sk -u admin:'<pw>' https://bmc.example.lab/redfish/v1/UpdateService/FirmwareInventory | jq .
```

Security-relevant features: TLS transport, session tokens (`X-Auth-Token`), role-based privileges (Administrator, Operator, ReadOnly, plus custom roles in newer schemas), account lockout policies, event subscriptions for alerting, and a standard update service. Redfish also defines a *Host Interface* that lets the OS reach the BMC's Redfish service over an internal virtual NIC; CVE-2024-54085 in AMI MegaRAC was exploitable through that path according to Eclypsium and NVD.

Dell's OpenManage Python SDK reference (v1.0) shows how vendor tooling abstracts several protocols, offering Redfish, WS-Management and SNMP back ends for the same operations; older Dell Lifecycle Controller 1.5 documentation (2011) is entirely WS-Management based. In practice a BMC may expose Redfish, IPMI, WS-Man, SNMP and a web UI simultaneously, and hardening one does not harden the others.

## Attestation and the BMC as a measurement agent

Modern platforms also use the BMC as an attestation broker. A 2021 Google presentation, "Redfish signed measurements" by Vidya Satyamsetti and Jeff Andersen, describes the BMC acting as an SPDM requester or TPM client, collecting signed measurements from device roots of trust over MCTP and surfacing them through Redfish to a remote verifier. This makes the BMC part of the platform's trust chain, which raises the stakes of its own integrity.

## IPMI vs Redfish

| Aspect | IPMI 2.0 over LAN | Redfish |
|---|---|---|
| Transport | UDP 623, RMCP+ | HTTPS (TCP 443) |
| Authentication | RAKP with HMAC; protocol discloses hash material | HTTP Basic or session token over TLS |
| Encryption | Optional per cipher suite | TLS |
| Data model | Fixed commands and records | Extensible JSON schemas |
| Main risks | Protocol design weaknesses, cipher 0, default accounts | Web/API bugs, auth bypass, weak TLS config |

## Worked example

*Execution status: not executed in this book's CI; verify locally.*

Map the interfaces your own BMC exposes.

1. From a management workstation on the BMC VLAN, run `nmap -sU -p 623 <bmc-ip>` and `nmap -sT -p 22,80,443,5900,623,161 <bmc-ip>` against your own BMC.
2. Query IPMI capabilities: `ipmitool -I lanplus -H <bmc-ip> -U <user> -P '<pw>' channel getciphers ipmi 1`.
3. Walk Redfish read-only: `/redfish/v1/`, `/Managers/<id>/NetworkProtocol` to list enabled services and ports.
4. From the host OS, check the local paths: `sudo ipmitool mc info` (KCS) and whether a Redfish host interface NIC exists.
5. Record each interface, its authentication method and whether it is needed.

## Common pitfalls

- Disabling IPMI over LAN in the web UI while leaving it enabled through another interface or default reset.
- Using self-signed TLS certificates and teaching operators to click through warnings.
- Ignoring SNMP and WS-Man services on the BMC.
- Assuming the host-side interfaces require authentication.
- Leaving virtual media and KVM enabled when not needed.

## Exercises

1. Build a table of every service on one of your BMCs, its port, protocol, authentication and whether you can disable it.
2. Write a script that pulls firmware inventory over Redfish from all your BMCs and outputs versions.
3. Compare the default enabled services on two BMC vendors or firmware versions you own.

## Further reading

- DMTF Redfish specification and schemas: <https://www.dmtf.org/standards/redfish>
- IPMI 2.0 specification (Intel, archived)
- OpenBMC project: <https://github.com/openbmc>
- V. Satyamsetti, J. Andersen, "Redfish signed measurements" (Google, 2021)
- Dell EMC OpenManage Python SDK v1.0 API Reference Guide; Dell Lifecycle Controller 1.5 Web Services Interface Guide (2011)
- Related books: **firmware-fuzzing-llm-peripherals**, **tee-attack-engineering**
