# Resources

## Standards and guidance

- DMTF Redfish: <https://www.dmtf.org/standards/redfish>
- IPMI 2.0 specification (Intel, archived)
- CISA and NSA, "Harden Baseboard Management Controllers" (June 2023): <https://www.cisa.gov/news-events/alerts/2023/06/14/cisa-and-nsa-release-joint-guidance-hardening-baseboard-management-controllers-bmcs>
- bettercrypto.org, *Applied Crypto Hardening* (2014)
- Caltech IMSS, "IPMI Information": <https://www.imss.caltech.edu/services/security/it-staff/ipmi-information>

## Tools

- ipmitool: <https://github.com/ipmitool/ipmitool>
- Nmap NSE scripts `ipmi-version`, `ipmi-cipher-zero`, `supermicro-ipmi-conf`: <https://nmap.org/nsedoc/>
- Metasploit `auxiliary/scanner/ipmi/ipmi_dumphashes`: <https://www.rapid7.com/db/modules/auxiliary/scanner/ipmi/ipmi_dumphashes>
- hashcat (mode 7300) and John the Ripper (RAKP format) for lab password audits
- binwalk, unblob for firmware extraction
- OpenBMC: <https://github.com/openbmc>

## Research and advisories

- F. Périgaud, A. Gazet, J. Czarny, "Subverting your server through its BMC: the HPE iLO4 case", SSTIC 2018: <https://airbus-seclab.github.io/ilo/>
- CVE-2013-4786 (IPMI RAKP), CVE-2017-12542 (iLO 4), CVE-2019-6260 (Pantsdown), CVE-2024-54085 (AMI MegaRAC): <https://nvd.nist.gov>
- Censys advisory, CVE-2024-54085: <https://censys.com/advisory/cve-2024-54085/>
- iLOBleed coverage: <https://therecord.media/threat-actor-uses-hp-ilo-rootkit-to-wipe-servers/>

## Books and talks

- A. Matrosov, E. Rodionov, S. Bratus, *Rootkits and Bootkits* (No Starch Press, 2019)
- J. Picolet, *Hash Crack: Password Cracking Manual*
- P. Calderon, *Nmap Network Exploration and Security Auditing Cookbook*
- R. Davis, *The Art of Network Penetration Testing* (Manning, 2020)
- S. Metcalf, "Exploiting Administrator Insecurities", DEF CON 26
- A. Ermolov, "Data-Only Attacks Against UEFI BIOS", ZeroNights 2021
- V. Satyamsetti, J. Andersen, "Redfish signed measurements" (Google, 2021)

## Related books in this wiki

- **validator-operations-security**, **blockchain-rpc-node-security**, **firmware-fuzzing-llm-peripherals**
