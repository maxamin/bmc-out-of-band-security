# A BMC Security Programme, Node Hosting and Research Directions

The earlier chapters covered the technical controls. This chapter turns them into a programme: who owns the management plane, how it is assessed, how incidents are handled, what changes for blockchain validators and other node operators who rent hardware, and where public research is heading.

## Ownership and inventory

BMCs fall between teams. Network teams own the VLAN, server teams own the hardware, security teams own monitoring, and nobody owns the firmware. Assign one owner for the management plane with authority over all three. The minimum inventory per server: BMC vendor, model, firmware version, IP, network placement, enabled services, account list, last configuration export, and the host it controls.

## Assessment cadence

| Activity | Frequency |
|---|---|
| External exposure check of your own ranges for BMC services | Continuous / weekly |
| Firmware version versus advisories | Weekly; immediately for KEV-listed issues |
| Configuration drift diff | Daily |
| Account review | Monthly |
| Internal penetration test including the management VLAN | Annually, authorised and scoped |
| Firmware image review (SBOM, signature) for new models | At procurement |

## Incident response

Treat a suspected BMC compromise as a compromise of the host and potentially of neighbouring BMCs.

1. Isolate the BMC port at the switch rather than relying on BMC settings.
2. Preserve logs already shipped off-device; do not trust the local log.
3. Assume host firmware may be modified. Plan to reflash both BMC and host firmware from vendor images through an independent path (hardware recovery jumper, vendor service tool, or replacement board).
4. Rotate every credential the BMC held, and every credential shared with other BMCs.
5. Rebuild the host OS after firmware is restored, not before.
6. Report to the vendor PSIRT with indicators; for suspected new vulnerabilities, follow coordinated disclosure.

## Node and validator hosting

Validators, RPC nodes and other key-holding infrastructure are often run on rented bare-metal servers or in colocation. The BMC changes the trust picture:

- **The provider has BMC access.** Staff or compromised provider tooling can reboot, mount media, or open the console of your server. Encrypt disks, avoid storing signing keys in plaintext on disk, and prefer remote signers or HSMs so that a console session or disk image does not yield keys. See **validator-operations-security** for remote signers and slashing protection.
- **Ask about the management network.** Is the BMC on a provider-only VLAN, or is it exposed to other tenants or the internet? Does the provider expose a web KVM to you, and how is that authenticated?
- **Reboots as an attack.** BMC power control can cause downtime, missed duties, or, worse, a failover in which two validator instances sign at once. Monitoring should alert on unexpected power cycles and boot-order changes.
- **Firmware state at hand-over.** For second-hand or reprovisioned servers, ask the provider for BMC factory reset and current firmware, or perform it yourself if you have access.
- **Own hardware.** If you colocate your own servers, apply this book's controls fully: dedicated BMC port on your own VLAN, no internet route, VPN or bastion access, unique credentials.

**blockchain-rpc-node-security** covers the RPC layer of the same infrastructure.

## Programme checklist

- [ ] Single owner for the management plane; RACI documented.
- [ ] Complete BMC inventory with firmware versions.
- [ ] No BMC service reachable from the internet; verified by external checks of your own ranges.
- [ ] Dedicated, unrouted management VLAN; access via MFA bastion only; inter-BMC filtering.
- [ ] IPMI over LAN disabled, or restricted to suites 3/17 with cipher 0 off and long unique passwords.
- [ ] Unique random credentials per BMC in a vault; default and anonymous accounts disabled.
- [ ] Directory or RADIUS integration with MFA where supported; break-glass account tested.
- [ ] Unneeded services disabled (SNMP v1/v2c, HTTP, SSDP, virtual media, Redfish host interface).
- [ ] Trusted TLS certificates; weak protocols disabled.
- [ ] Firmware patch SLA; PSIRT subscriptions for OEM and upstream firmware vendors.
- [ ] Signed updates and anti-rollback enabled; hardware root of trust used where available.
- [ ] Remote logging and Redfish events to SIEM; alerts on logins, accounts, firmware, media, power.
- [ ] Daily configuration drift checks.
- [ ] Incident runbook for BMC compromise with independent reflash path.
- [ ] Procurement checklist covering firmware lineage, signing, RoT, and default credentials.
- [ ] For hosted nodes: provider BMC practices documented and contractually covered.

## Research directions

Open areas in public work include: attestation of BMC firmware itself, not just host measurements collected by the BMC (Google's 2021 "Redfish signed measurements" work shows the BMC as an SPDM requester; the BMC's own integrity is the next question); memory-safe rewrites of BMC services (OpenBMC components in Rust and C++ with modern practices); fuzzing of Redfish, IPMI and KVM services in emulation, where firmware rehosting research applies (see **firmware-fuzzing-llm-peripherals**); faster downstream patch propagation when one upstream stack underlies many brands; and safe multi-tenant bare-metal designs where tenants cannot reach the BMC from the host at all.

## Worked example

*Execution status: not executed in this book's CI; verify locally.*

Run a tabletop exercise for a BMC compromise on a validator host you operate.

1. Scenario: your monitoring shows an unexpected virtual-media mount and reboot on validator host V1, followed by an unfamiliar BMC account.
2. Walk through the incident steps above with the team. Who isolates the switch port? Where are BMC logs stored off-device? Does anyone know the independent reflash procedure for this model?
3. Check the signing setup: would a disk image or console session have exposed keys? Is slashing protection in place if V1 is restored while a backup is active?
4. Record the gaps and assign owners and dates.
5. Repeat the scenario for a rented server where the provider holds BMC access, and draft the questions to send the provider.

## Common pitfalls

- Programmes that cover BMC firmware but not host-side interfaces.
- Incident plans that reinstall the OS while leaving BMC and BIOS firmware untouched.
- Assuming hosting providers isolate BMCs without asking.
- Storing validator keys unencrypted on servers where others hold console access.
- Running internal BMC tests without written scope and change control.

## Exercises

1. Score your environment against the checklist and produce a prioritised remediation plan.
2. Write a procurement questionnaire for server vendors and hosting providers based on this book.
3. Run the tabletop above and publish the action items internally.

## Further reading

- CISA and NSA, "Harden Baseboard Management Controllers" (2023)
- A. Matrosov, E. Rodionov, S. Bratus, *Rootkits and Bootkits* (2019)
- V. Satyamsetti, J. Andersen, "Redfish signed measurements" (Google, 2021)
- OpenBMC security documentation: <https://github.com/openbmc/docs>
- Related books: **validator-operations-security**, **blockchain-rpc-node-security**, **firmware-fuzzing-llm-peripherals**
